# File Uploader - ACE Method Example

## Phase 1: START - Define Your Project

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Project Identity
**What I'm Building:**
[A modern drag-and-drop file upload system with real-time progress tracking, chunked uploads for large files, automatic retry logic, and support for multiple cloud storage providers]

**The 5-Minute Pitch:**
[DropZone Pro is an enterprise-grade file upload solution that transforms the file upload experience. It features drag-and-drop interface, resumable uploads for files up to 5GB, real-time progress with speed calculations, automatic virus scanning, image optimization on-the-fly, and seamless integration with S3, Azure Blob, and Google Cloud Storage. Built with React and Node.js, it handles network interruptions gracefully and provides detailed upload analytics.]

#### The Reality Check

**Who Will Use This:**
[SaaS applications needing reliable file uploads, Content management systems, E-learning platforms with video uploads, Healthcare systems handling medical images, Creative agencies managing large design files]

**The Problem It Solves:**
[Eliminates failed uploads for large files, Provides professional upload experience, Handles unreliable network connections, Reduces storage costs with optimization, Ensures security with virus scanning]

**Why They'll Choose This:**
[Resumable uploads save bandwidth, Chunked upload for 5GB+ files, Multi-cloud storage support, Built-in image/video optimization, Real-time progress with ETA, Automatic retry on failure]

#### Constraints & Resources

**Technical Constraints:**
[Support files up to 5GB per upload, Handle 1000 concurrent uploads, Work on connections as slow as 1Mbps, Compatible with all modern browsers, 4GB RAM minimum for server]

**Time Investment:**
[3 weeks for core upload functionality, 2 weeks for chunking and resume, 1 week for progress tracking UI, 1 week for cloud integrations, 1 week for optimization features]

**Financial Reality:**
[S3 storage: $23/TB/month, CloudFront CDN: $85/TB transferred, Server hosting: $100/month, Virus scanning API: $50/month, Image optimization: $30/month, Total: ~$300/month base]

#### Scope Definition

**Core Features (MVP):**
- Drag-and-drop file selection
- Multi-file upload support
- Real-time progress bars
- Basic file validation
- S3 integration
- Upload cancellation

**Future Expansions:**
- Chunked uploads for large files
- Resume interrupted uploads
- Multiple storage providers
- Automatic image optimization
- Video transcoding
- Virus scanning integration

**Out of Scope:**
- File preview/editing
- Document conversion
- OCR capabilities
- Collaborative features
- Desktop application

---

## Phase 2: ANALYZE - Deep Requirements Discovery

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Problem Deep Dive

**The Core Challenge:**
[Creating a file upload system that handles large files reliably over unstable connections, provides accurate progress feedback, manages memory efficiently for concurrent uploads, and integrates seamlessly with multiple cloud providers while maintaining security and performance]

**Current Pain Points:**
[Browser timeouts on large uploads, Lost progress when connection drops, No visibility into upload status, Memory crashes with multiple files, Inconsistent experience across browsers, Security vulnerabilities with direct uploads]

**Success Metrics:**
[99% upload success rate, Support 5GB single file uploads, Handle 100 concurrent uploads, Resume within 2 seconds of connection restore, Memory usage under 500MB for 10 uploads]

#### User Journey Mapping

**Primary User Flow:**
[User drags files to dropzone → Files validated for type/size → Upload starts with progress → Network interruption handled → Upload auto-resumes → Files processed (optimization/scanning) → Success notification with file URL]

**Critical Touchpoints:**
[Drag-and-drop responsiveness, Validation error clarity, Progress accuracy, Network error handling, Resume experience, Completion confirmation]

**Failure Points:**
[Browser crashes on large files, Confusing error messages, Progress bar stuck at 99%, Lost uploads need restart, No indication of retry attempts, Timeout without explanation]

#### System Boundaries

**What We'll Build:**
- React component library for UI
- Node.js upload server
- Chunking algorithm for large files
- Resume logic with state persistence
- Progress tracking system
- Multi-cloud storage abstraction
- File validation framework
- Upload queue management

**What We'll Integrate:**
- AWS S3 for storage
- Azure Blob Storage
- Google Cloud Storage
- ClamAV for virus scanning
- Sharp for image optimization
- FFmpeg for video processing

**What We Won't Touch:**
- File editing capabilities
- Permission management
- File sharing features
- Storage quota management
- Billing integration

#### Success Criteria

**Launch Requirements:**
[5GB file upload tested successfully, Resume feature works across browser restart, Memory usage stays under 500MB, 99% success rate in testing, Works on 3G connections]

**Quality Benchmarks:**
[Upload speed 90% of connection capacity, Progress updates every 100ms, Memory per file under 50MB, Retry success rate > 95%, Browser compatibility 95%+]

**Adoption Targets:**
[1M files uploaded in month 1, 50TB total storage by month 3, 1000 active implementations, 4.5/5 developer satisfaction, 10 enterprise customers]

---

## Phase 3: CREATE - Implementation Problem Solving

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Current Context

**What I'm Building:**
[Implementing the chunked upload system that splits large files into manageable pieces, tracks upload state for resume capability, and handles memory efficiently without crashing the browser]

**Where I'm Stuck:**
[The file chunking is causing memory spikes that crash the browser with files over 1GB. Also having issues with chunk ordering on the server side when uploads arrive out of sequence. The resume logic can't properly identify which chunks were already uploaded.]

**What I've Tried:**
[Used FileReader API with slicing, tried Web Workers for chunking, implemented server-side chunk assembly, used IndexedDB for state persistence, attempted streaming with ReadableStream API]

#### Code State

**Relevant Code:**
```javascript
// Current problematic chunking implementation
class ChunkedUploader {
  constructor(file, options = {}) {
    this.file = file;
    this.chunkSize = options.chunkSize || 5 * 1024 * 1024; // 5MB
    this.chunks = [];
    this.uploadedChunks = new Set();
  }

  // This loads entire file into memory - crashes on large files
  async prepareChunks() {
    const totalChunks = Math.ceil(this.file.size / this.chunkSize);
    
    for (let i = 0; i < totalChunks; i++) {
      const start = i * this.chunkSize;
      const end = Math.min(start + this.chunkSize, this.file.size);
      
      // Problem: All chunks created at once
      const chunk = this.file.slice(start, end);
      const arrayBuffer = await chunk.arrayBuffer(); // Memory spike!
      
      this.chunks.push({
        index: i,
        data: arrayBuffer,
        hash: await this.calculateHash(arrayBuffer),
        size: end - start
      });
    }
  }

  // Server-side chunk assembly with ordering issues
  async handleChunkUpload(req, res) {
    const { uploadId, chunkIndex, totalChunks } = req.body;
    const chunkData = req.file.buffer;
    
    // Race condition: Multiple chunks can arrive simultaneously
    const uploadDir = path.join('temp', uploadId);
    if (!fs.existsSync(uploadDir)) {
      fs.mkdirSync(uploadDir, { recursive: true });
    }
    
    // No verification of chunk integrity
    const chunkPath = path.join(uploadDir, `chunk_${chunkIndex}`);
    fs.writeFileSync(chunkPath, chunkData);
    
    // Inefficient check for completion
    const files = fs.readdirSync(uploadDir);
    if (files.length === totalChunks) {
      // Assembly without proper ordering
      const chunks = files
        .sort((a, b) => {
          // This sort is fragile
          const aIndex = parseInt(a.split('_')[1]);
          const bIndex = parseInt(b.split('_')[1]);
          return aIndex - bIndex;
        })
        .map(file => fs.readFileSync(path.join(uploadDir, file)));
      
      const finalFile = Buffer.concat(chunks);
      // Missing: Verify final file integrity
      await this.uploadToS3(uploadId, finalFile);
    }
    
    res.json({ success: true, chunkIndex });
  }

  // Resume logic that doesn't work properly
  async resumeUpload() {
    // This doesn't account for partially uploaded chunks
    const uploadState = await this.getUploadState();
    
    for (let i = 0; i < this.chunks.length; i++) {
      if (uploadState.completedChunks.includes(i)) {
        continue; // Skip completed chunks
      }
      
      // No retry logic for failed chunks
      await this.uploadChunk(i);
    }
  }
}

// Frontend progress tracking with issues
const FileUploadComponent = () => {
  const [progress, setProgress] = useState({});
  
  const handleUpload = async (file) => {
    const uploader = new ChunkedUploader(file);
    
    // This blocks the UI
    await uploader.prepareChunks();
    
    // Progress calculation is inaccurate
    uploader.on('chunkComplete', (chunkIndex, totalChunks) => {
      setProgress(prev => ({
        ...prev,
        [file.name]: (chunkIndex + 1) / totalChunks * 100
      }));
    });
    
    await uploader.start();
  };
};
```

**Error Messages:**
["RangeError: Array buffer allocation failed" for large files, "Maximum call stack exceeded" during chunk assembly, "EMFILE: too many open files" on server, Browser becomes unresponsive during upload]

**What's Working:**
[Small file uploads work perfectly, Drag-and-drop interface is smooth, S3 integration is solid, Basic progress tracking functional, File validation working well]

#### Environment Details

**Tech Stack:**
[React 18 with TypeScript, Node.js 18 with Express, AWS SDK v3, Multer for multipart, Socket.io for progress, IndexedDB for state, Web Workers for processing]

**Constraints:**
[Browser memory limit ~2GB, Max request size 100MB, S3 multipart requires 5MB+ chunks, Must support Safari 14+, Server has 8GB RAM]

**Dependencies:**
[AWS S3 with multipart upload API, CloudFront for signed URLs, Redis for upload state, PM2 for process management, Nginx for load balancing]

#### Specific Questions

**Technical Challenges:**
[How to chunk files without loading into memory? Best approach for chunk ordering verification? How to handle partial chunk uploads? Should we use service workers for resilience?]

**Design Decisions:**
[Stream processing vs chunk-based approach? Optimal chunk size for various connections? Client-side vs server-side chunk assembly? How to prevent duplicate uploads?]

**Implementation Approach:**
[Considering Blob.stream() API for memory efficiency, Evaluating tus protocol for resumable uploads, Thinking about WebRTC for P2P transfers, Exploring Cloudflare R2 for storage]

---

## Phase 4: EVALUATE - Testing & Quality Assurance

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Implementation Status

**What's Complete:**
- Drag-and-drop interface
- Multi-file selection
- Basic progress tracking
- File type validation
- S3 integration
- Small file uploads (<100MB)
- Basic error handling
- Upload cancellation
- File metadata extraction

**What's In Progress:**
- Chunked upload optimization
- Resume functionality
- Memory management fixes
- Progress accuracy improvements
- Multi-cloud support

**What's Pending:**
- Virus scanning integration
- Image optimization
- Video transcoding
- Advanced analytics
- Batch upload API

#### The Reality Check

**Known Issues:**
[Browser crashes with files over 1GB, Chunk assembly fails randomly on server, Resume doesn't work after browser restart, Progress jumps from 90% to 100%, Memory leaks in long upload sessions]

**Performance Bottlenecks:**
[File reading blocks UI thread, Chunk hashing takes too long, Server chunk assembly uses too much RAM, S3 multipart initialization slow, Progress events flood the UI]

**Security Concerns:**
[No file content validation, Missing virus scanning, Unsigned upload URLs exposed, Path traversal in chunk assembly, No rate limiting on uploads]

#### Quality Assessment

**Test Coverage:**
[Unit tests: 70% coverage, Integration tests: 45% coverage, E2E tests: 30% coverage, Browser testing: Chrome/Firefox only, Load testing: Up to 10 concurrent uploads]

**Code Review Findings:**
[Memory management needs refactoring, Error handling inconsistent, No cleanup of failed uploads, Missing TypeScript types, Hard-coded configuration values]

**External Feedback:**
[Users report browser freezing, Progress bar not accurate, Resume feature confusing, No clear error messages, Mobile experience poor]

#### The Launch Checklist

**Must-Fix Before Launch:**
- Fix memory issues with large files
- Implement proper chunk ordering
- Make resume work reliably
- Add security headers
- Implement rate limiting
- Fix progress accuracy
- Add virus scanning
- Clean up temp files

**Should-Fix If Time:**
- Optimize chunk size dynamically
- Add bandwidth throttling
- Implement retry backoff
- Improve error messages
- Add upload analytics
- Create React hooks
- Support more browsers
- Add integration tests

**Can Wait Until Post-Launch:**
- Video transcoding
- Image optimization
- Advanced analytics
- Batch upload API
- Desktop app
- Collaboration features
- Custom storage providers
- Webhooks

---

## Phase 5: COMMIT - Deployment & Operations

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Launch Readiness

**Deployment Plan:**
[Deploy frontend to Vercel with CDN, Upload servers on AWS ECS with auto-scaling, S3 buckets in multiple regions, CloudFront for global distribution, ElastiCache for upload state, Route53 for geo-routing]

**Rollback Strategy:**
[Vercel instant rollback for frontend, Blue-green ECS deployment, Feature flags for new features, S3 versioning enabled, Database migration rollbacks ready]

**Success Metrics:**
[99% upload success rate, 5GB file upload capability, 100 concurrent uploads supported, <2 second resume time, Global availability 99.9%]

#### Deployment Details

**Infrastructure Checklist:**
- Vercel project with env vars
- ECS task definitions updated
- S3 buckets with CORS configured
- CloudFront distributions ready
- ElastiCache cluster deployed
- Auto-scaling policies set
- Health checks configured
- SSL certificates valid
- WAF rules for security
- Monitoring alerts ready

**Configuration Management:**
[Environment variables in Vercel/AWS, S3 credentials in Secrets Manager, Feature flags in LaunchDarkly, Infrastructure defined in Terraform, Config validation on startup]

**Monitoring Setup:**
[CloudWatch for infrastructure, Sentry for frontend errors, DataDog for upload metrics, S3 metrics for storage, Custom dashboard for upload analytics, PagerDuty for incidents]

#### Post-Launch Planning

**Day 1 Priorities:**
- Monitor upload success rates
- Check memory usage patterns
- Verify resume functionality
- Watch for security alerts
- Monitor S3 costs
- Track browser errors
- Review server performance
- Respond to user feedback

**Week 1 Goals:**
- Analyze upload patterns
- Optimize chunk sizes
- Fix discovered bugs
- Improve error messages
- Create usage documentation
- Set up cost alerts
- Plan feature updates
- Gather user testimonials

**Month 1 Targets:**
- 1M successful uploads
- 100TB data transferred
- 99% success rate maintained
- 500 active integrations
- 5 enterprise customers
- SDK libraries released
- Community forum launched
- Version 2.0 planned

#### Retrospective

**What Went Well:**
[Drag-and-drop UX exceeded expectations, S3 integration was straightforward, Progress tracking loved by users, Chunking algorithm finally optimized, Security audit passed first time]

**Lessons Learned:**
[Browser memory limits stricter than expected, Chunk assembly needed distributed approach, Resume logic more complex than anticipated, Testing with real network conditions crucial, Safari compatibility requires extra work]

**Team Victories:**
[Solved the large file upload problem, Created butter-smooth upload experience, Zero security vulnerabilities found, Performance goals exceeded, Amazing developer documentation created]