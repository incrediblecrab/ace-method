# Data Migrator - ACE Method Example

## Phase 1: START - Define Your Project

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Project Identity
**What I'm Building:**
[A robust database migration tool that handles schema changes, data transformations, with support for rollbacks, zero-downtime migrations, and cross-database platform migrations]

**The 5-Minute Pitch:**
[MigrateDB Pro is an enterprise-grade database migration platform that makes database changes safe and reversible. It features automatic migration generation, intelligent rollback capabilities, zero-downtime deployments, data validation, and migration testing. Built with Go for performance, it supports PostgreSQL, MySQL, MongoDB, and SQL Server while providing detailed migration history, dry-run capabilities, and team collaboration features for database change management.]

#### The Reality Check

**Who Will Use This:**
[DevOps teams managing production databases, Developers needing safe schema updates, DBAs coordinating complex migrations, Companies doing platform migrations, Teams requiring audit trails]

**The Problem It Solves:**
[Eliminates risky manual database changes, Provides rollback safety net, Enables zero-downtime deployments, Tracks all database changes, Prevents data loss during migrations]

**Why They'll Choose This:**
[Automatic rollback generation, Support for multiple databases, Zero-downtime migrations, Comprehensive testing tools, Git-like version control for schemas]

#### Constraints & Resources

**Technical Constraints:**
[Must handle 100TB+ databases, Support online migrations, Work with 5-year old database versions, Handle 10k tables per database, Complete rollback under 1 minute]

**Time Investment:**
[3 weeks for core migration engine, 2 weeks for rollback system, 2 weeks for multi-DB support, 1 week for testing framework, 1 week for CLI and UI]

**Financial Reality:**
[Development servers: $200/month, Test database instances: $300/month, CI/CD infrastructure: $100/month, Monitoring tools: $50/month, Documentation hosting: $20/month, Total: ~$670/month]

#### Scope Definition

**Core Features (MVP):**
- Schema migration execution
- Automatic rollback scripts
- Migration history tracking
- PostgreSQL and MySQL support
- CLI tool
- Basic validation

**Future Expansions:**
- Zero-downtime migrations
- Data transformation pipelines
- Cross-database migrations
- Visual migration builder
- Team collaboration
- Migration analytics

**Out of Scope:**
- Database administration
- Backup management
- Performance tuning
- Query optimization
- Database monitoring

---

## Phase 2: ANALYZE - Deep Requirements Discovery

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Problem Deep Dive

**The Core Challenge:**
[Creating a migration system that can safely modify production databases without downtime, generate accurate rollback scripts automatically, handle complex data transformations, validate data integrity throughout the process, and work across different database platforms while maintaining ACID compliance]

**Current Pain Points:**
[Manual migrations cause production outages, No reliable rollback when things go wrong, Difficulty tracking who changed what, Data corruption during migrations, Different tools for different databases]

**Success Metrics:**
[Zero data loss in 1000 migrations, Rollback success rate 99.9%, Support migrations on 1TB+ databases, Migration execution under 1 hour, 100% ACID compliance maintained]

#### User Journey Mapping

**Primary User Flow:**
[Developer writes schema change → Tool generates migration files → Reviews migration plan → Runs dry-run test → Executes on staging → Validates results → Deploys to production → Monitors progress → Rollback if needed]

**Critical Touchpoints:**
[Migration file generation accuracy, Dry-run results clarity, Progress monitoring visibility, Rollback trigger simplicity, Validation report completeness]

**Failure Points:**
[Generated migrations miss dependencies, Dry-run doesn't catch all issues, Production behavior differs from staging, Rollback fails due to data changes, Validation misses data corruption]

#### System Boundaries

**What We'll Build:**
- Migration file generator
- Execution engine with transactions
- Rollback script generator
- Progress monitoring system
- Data validation framework
- Multi-database abstraction
- CLI and Web UI
- Migration testing tools

**What We'll Integrate:**
- PostgreSQL, MySQL, MongoDB, SQL Server
- Git for version control
- Slack for notifications
- Prometheus for metrics
- S3 for migration backups
- Kubernetes for orchestration

**What We Won't Touch:**
- Database performance optimization
- Query plan analysis
- Backup and restore
- Replication management
- Database security

#### Success Criteria

**Launch Requirements:**
[Successfully migrate 10 production databases, Zero-downtime migration tested, Rollback tested on all scenarios, Support 4 database platforms, 100% test coverage on core]

**Quality Benchmarks:**
[Migration generation < 1 second, Dry-run accuracy 100%, Rollback generation 100% accurate, Large table migration < 1 hour, Memory usage < 1GB]

**Adoption Targets:**
[1000 migrations run month 1, 100 active teams, 50TB data migrated, Zero data loss incidents, 10 enterprise customers]

---

## Phase 3: CREATE - Implementation Problem Solving

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Current Context

**What I'm Building:**
[Implementing the zero-downtime migration system that can alter large tables without locking, using techniques like shadow tables, triggers for data sync, and atomic table swaps]

**Where I'm Stuck:**
[The trigger-based sync is causing performance degradation on high-traffic tables. Also having issues with foreign key constraints during the table swap. The rollback mechanism doesn't work properly when new data is added during migration.]

**What I've Tried:**
[Implemented shadow table approach, used triggers for real-time sync, tried batched data copying, attempted online schema change algorithms, used table renaming for atomic swap]

#### Code State

**Relevant Code:**
```go
// Current problematic zero-downtime implementation
type OnlineMigration struct {
    source      *sql.DB
    shadowTable string
    triggers    []string
    batchSize   int
}

// This approach causes performance issues
func (m *OnlineMigration) createSyncTriggers(tableName string) error {
    // Trigger slows down writes significantly
    insertTrigger := fmt.Sprintf(`
        CREATE TRIGGER %s_insert_sync
        AFTER INSERT ON %s
        FOR EACH ROW
        BEGIN
            INSERT INTO %s SELECT NEW.*;
        END
    `, tableName, tableName, m.shadowTable)
    
    _, err := m.source.Exec(insertTrigger)
    if err != nil {
        return err
    }
    
    // Update trigger is even slower
    updateTrigger := fmt.Sprintf(`
        CREATE TRIGGER %s_update_sync
        AFTER UPDATE ON %s
        FOR EACH ROW
        BEGIN
            DELETE FROM %s WHERE id = NEW.id;
            INSERT INTO %s SELECT NEW.*;
        END
    `, tableName, tableName, m.shadowTable, m.shadowTable)
    
    _, err = m.source.Exec(updateTrigger)
    return err
}

// Foreign key handling is broken
func (m *OnlineMigration) performTableSwap(oldTable, newTable string) error {
    tx, err := m.source.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()
    
    // This fails with foreign key constraints
    tempName := fmt.Sprintf("%s_old_%d", oldTable, time.Now().Unix())
    
    // Atomic rename doesn't work with FKs
    _, err = tx.Exec(fmt.Sprintf("RENAME TABLE %s TO %s, %s TO %s",
        oldTable, tempName, newTable, oldTable))
    if err != nil {
        return err
    }
    
    return tx.Commit()
}

// Rollback doesn't account for new data
type RollbackGenerator struct {
    migrations map[string]*Migration
    snapshots  map[string]*TableSnapshot
}

func (r *RollbackGenerator) generateRollback(migration *Migration) (*Migration, error) {
    rollback := &Migration{
        ID: fmt.Sprintf("rollback_%s", migration.ID),
    }
    
    for _, change := range migration.Changes {
        switch change.Type {
        case "ADD_COLUMN":
            // This doesn't preserve data added to the column
            rollback.Changes = append(rollback.Changes, Change{
                Type:   "DROP_COLUMN",
                Column: change.Column,
            })
        case "MODIFY_COLUMN":
            // No data type compatibility check
            rollback.Changes = append(rollback.Changes, Change{
                Type:     "MODIFY_COLUMN",
                Column:   change.Column,
                DataType: change.OldDataType,
            })
        }
    }
    
    return rollback, nil
}

// Progress tracking missing key metrics
type MigrationProgress struct {
    TablesCompleted int
    RowsCopied      int64
    StartTime       time.Time
    mutex           sync.Mutex
}

func (p *MigrationProgress) updateProgress(table string, rows int64) {
    p.mutex.Lock()
    defer p.mutex.Unlock()
    
    p.TablesCompleted++
    p.RowsCopied += rows
    
    // No ETA calculation or rate limiting info
    log.Printf("Progress: %d tables, %d rows copied", 
        p.TablesCompleted, p.RowsCopied)
}
```

**Error Messages:**
["Deadlock found when trying to get lock", "Cannot drop column: needed in foreign key constraint", "Trigger slowed INSERT by 10x", "Lost 1000 rows during table swap", "Rollback failed: incompatible data types"]

**What's Working:**
[Basic migrations work perfectly, Small table migrations fast, Migration history tracking solid, Dry-run catches most issues, Multi-database support functional]

#### Environment Details

**Tech Stack:**
[Go 1.21, PostgreSQL 14, MySQL 8, MongoDB 6, SQL parsing library, Cobra for CLI, React for web UI, gRPC for API]

**Constraints:**
[Must not lock tables > 1 minute, Support tables with 1B+ rows, Maintain referential integrity, Rollback must complete < 5 minutes, Memory limit 2GB]

**Dependencies:**
[Database connection pools, Kubernetes for job orchestration, Redis for distributed locks, S3 for backup storage, Prometheus for monitoring]

#### Specific Questions

**Technical Challenges:**
[How to sync data without triggers? Best approach for foreign key constraints? How to estimate migration duration accurately? Should we use logical replication?]

**Design Decisions:**
[Trigger-based vs log-based sync? How to handle very large tables? Where to run migration jobs? How to coordinate distributed migrations?]

**Implementation Approach:**
[Considering binlog parsing for MySQL, Logical replication for PostgreSQL, Thinking about pt-online-schema-change patterns, Exploring blue-green deployments]

---

## Phase 4: EVALUATE - Testing & Quality Assurance

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Implementation Status

**What's Complete:**
- Core migration engine
- Basic rollback generation
- PostgreSQL and MySQL support
- CLI tool with all commands
- Migration history tracking
- Simple web UI
- Dry-run functionality
- Integration tests
- Documentation

**What's In Progress:**
- Zero-downtime improvements
- Foreign key handling
- Performance optimization
- MongoDB support
- Advanced rollback logic

**What's Pending:**
- SQL Server support
- Visual migration builder
- Team collaboration
- Migration analytics
- Cloud deployment

#### The Reality Check

**Known Issues:**
[Triggers slow down production by 5-10x, Foreign key constraints block table swaps, Large table migrations take 10+ hours, Rollback fails with data type changes, Memory usage spikes on big tables]

**Performance Bottlenecks:**
[Trigger overhead unacceptable, Full table scans for validation, No parallel processing, Shadow table creation slow, Progress tracking queries expensive]

**Security Concerns:**
[Database credentials in plain text, No audit log encryption, Missing role-based access, SQL injection in dynamic queries, No data masking for dev/test]

#### Quality Assessment

**Test Coverage:**
[Unit tests: 85% coverage, Integration tests: 70% coverage, Database tests: All 4 platforms, Performance tests: Up to 1M rows, Rollback tests: 20 scenarios]

**Code Review Findings:**
[SQL building needs parameterization, Error handling inconsistent, No connection retry logic, Missing transaction boundaries, Resource cleanup issues]

**External Feedback:**
[Users want better progress visibility, Zero-downtime too slow for production, Need better error messages, Want migration scheduling, Missing data validation options]

#### The Launch Checklist

**Must-Fix Before Launch:**
- Fix trigger performance issues
- Handle foreign keys properly
- Encrypt sensitive data
- Add connection pooling
- Implement parallel processing
- Fix memory leaks
- Add comprehensive logging
- Create runbooks

**Should-Fix If Time:**
- Add progress ETA
- Implement batching options
- Create migration templates
- Add validation rules
- Improve UI performance
- Add webhook notifications
- Create video tutorials
- Set up status page

**Can Wait Until Post-Launch:**
- SQL Server support
- Visual builder
- Analytics dashboard
- Cloud marketplace
- Advanced scheduling
- Data masking
- Compliance reports
- API v2

---

## Phase 5: COMMIT - Deployment & Operations

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Launch Readiness

**Deployment Plan:**
[Deploy CLI via Homebrew and package managers, Web UI on Kubernetes, API servers with auto-scaling, PostgreSQL for metadata, Redis for distributed locking, S3 for migration archives]

**Rollback Strategy:**
[Versioned CLI releases in GitHub, Kubernetes deployment history, Database backup before changes, Feature flags for risky features, Emergency runbooks prepared]

**Success Metrics:**
[1000 successful migrations month 1, Zero data loss incidents, 99% rollback success rate, Support 1TB+ databases, 100 active teams using tool]

#### Deployment Details

**Infrastructure Checklist:**
- GitHub releases configured
- Homebrew formula ready
- Docker images built
- Kubernetes manifests
- RDS instance prepared
- Redis cluster ready
- S3 buckets created
- Monitoring configured
- Documentation site live
- Support channels ready

**Configuration Management:**
[CLI configs in ~/.migratedb/, Server configs in Kubernetes secrets, Database creds in HashiCorp Vault, Feature flags in environment, Monitoring in Prometheus]

**Monitoring Setup:**
[Custom metrics for migrations, Duration and success tracking, Database load monitoring, Error rate dashboards, Slack alerts for failures, On-call rotation setup]

#### Post-Launch Planning

**Day 1 Priorities:**
- Monitor first migrations
- Track error rates
- Check performance metrics
- Verify rollback works
- Monitor resource usage
- Review user feedback
- Update documentation
- Respond to issues

**Week 1 Goals:**
- Analyze usage patterns
- Optimize slow operations
- Fix reported bugs
- Create FAQ section
- Host office hours
- Gather testimonials
- Plan v1.1 features
- Review architecture

**Month 1 Targets:**
- 10k migrations executed
- 100TB data migrated
- 500 active users
- 99.9% success rate
- 20 enterprise trials
- 5 case studies published
- Community forum active
- First conference talk

#### Retrospective

**What Went Well:**
[Rollback generation saved multiple production databases, Multi-database support architecture proved flexible, Dry-run feature caught countless issues, CLI UX received amazing feedback, Performance optimizations exceeded targets]

**Lessons Learned:**
[Zero-downtime harder than expected on busy databases, Foreign keys need special consideration, Progress tracking crucial for long migrations, Documentation quality directly impacts adoption, Community building starts before launch]

**Team Victories:**
[Successfully migrated 50TB production database, Achieved true zero-downtime on PostgreSQL, Built most comprehensive rollback system, Created migration testing framework, Fostered active open source community]