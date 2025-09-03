# Multi-Source Content Processing Specification

Detailed technical specifications for processing web URLs, YouTube videos, text files, and hybrid combinations into optimized flashcards with AI quality assessment and source attribution.

## Overview

The multi-source content processing system transforms diverse technical learning materials into personalized flashcards while maintaining full source attribution and quality metrics. This enables engineers to convert any learning material into spaced repetition format.

## Content Source Types

### 1. URL Content Extraction 🌐

#### Supported Content Types
- **Documentation Sites**: Official framework/library documentation (React, Angular, Vue, etc.)
- **Technical Blogs**: Medium, Dev.to, personal engineering blogs
- **Stack Overflow**: Questions, answers, and explanations
- **GitHub**: README files, wiki pages, issue discussions
- **Learning Platforms**: Tutorial sites, coding bootcamp materials
- **News Sites**: Technical news articles and announcements

#### Technical Implementation

##### Web Scraping Engine
```typescript
interface URLExtractionService {
  extractContent(url: string, options: ExtractionOptions): Promise<ExtractedContent>;
  validateURL(url: string): Promise<URLValidationResult>;
  getMetadata(url: string): Promise<ContentMetadata>;
}

interface ExtractionOptions {
  extractionType: 'full' | 'summary' | 'key_points';
  includeImages: boolean;
  includeCodeBlocks: boolean;
  maxWordCount: number;
  preserveFormatting: boolean;
}

interface ExtractedContent {
  mainContent: string;
  title: string;
  author?: string;
  publishDate?: string;
  codeBlocks: Array<{
    language: string;
    code: string;
    context: string;
  }>;
  images?: Array<{
    url: string;
    alt: string;
    caption?: string;
  }>;
  metadata: ContentMetadata;
}
```

##### Content Processing Pipeline
1. **URL Validation**: Check accessibility, content type, and size limits
2. **Content Extraction**: Use headless browser or API scraping based on site
3. **Cleaning & Filtering**: Remove navigation, ads, and irrelevant content
4. **Structure Analysis**: Identify headings, code blocks, lists, and key sections
5. **Content Optimization**: Format for AI processing and card generation

##### Supported Extraction Methods
```typescript
interface ExtractionStrategy {
  // Method 1: Headless Browser (for dynamic content)
  puppeteerExtraction: {
    waitForSelector: string;
    removeSelectors: string[];
    scrollBehavior: 'none' | 'lazy' | 'full';
  };
  
  // Method 2: API Integration (for supported sites)
  apiExtraction: {
    provider: 'github' | 'stackoverflow' | 'medium';
    endpoint: string;
    authentication?: string;
  };
  
  // Method 3: HTML Parsing (for static content)
  htmlParsing: {
    contentSelector: string;
    titleSelector: string;
    authorSelector?: string;
    dateSelector?: string;
  };
}
```

#### API Specification
```typescript
POST /api/content/extract-url
Authorization: Bearer <jwt_token>
Content-Type: application/json

Request Body:
{
  url: string;
  extractionType: 'full' | 'summary' | 'key_points';
  preferences?: {
    includeCodeBlocks: boolean;
    maxWordCount: number;
    focusAreas?: string[]; // e.g., ["API", "examples", "best practices"]
  };
}

Response:
{
  success: boolean;
  extractedContent: {
    mainContent: string;
    title: string;
    author?: string;
    publishDate?: string;
    wordCount: number;
    codeBlockCount: number;
    estimatedReadingTime: number; // minutes
  };
  metadata: {
    domain: string;
    contentType: 'documentation' | 'blog' | 'tutorial' | 'reference';
    lastUpdated?: string;
    language: string;
    difficulty: 'beginner' | 'intermediate' | 'advanced';
  };
  processingDetails: {
    extractionMethod: 'puppeteer' | 'api' | 'html_parsing';
    processingTime: number;
    contentQuality: number; // 0-1 quality assessment
  };
}
```

#### Error Handling & Edge Cases
- **Rate Limiting**: Implement exponential backoff for repeated requests
- **Content Size Limits**: Maximum 100KB extracted content per URL
- **Dynamic Content**: Handle JavaScript-rendered pages with timeout limits
- **Access Restrictions**: Graceful handling of paywalls, login requirements, geo-blocking
- **Malformed URLs**: URL validation and sanitization before processing

### 2. YouTube Video Processing 📺

#### Content Extraction Capabilities
- **Transcript Extraction**: Automatic closed caption retrieval in multiple languages
- **Key Moment Detection**: AI-powered identification of important segments
- **Chapter Processing**: Extract content based on video chapters or timestamps
- **Slide Recognition**: OCR processing for presentation-style videos
- **Speaker Identification**: Multiple speaker handling for interviews/panels

#### Technical Implementation

##### YouTube API Integration
```typescript
interface YouTubeProcessingService {
  getVideoMetadata(videoId: string): Promise<VideoMetadata>;
  extractTranscript(videoId: string, options: TranscriptOptions): Promise<Transcript>;
  identifyKeyMoments(transcript: string, metadata: VideoMetadata): Promise<KeyMoment[]>;
  processVideoSegments(videoId: string, segments: TimeSegment[]): Promise<ProcessedSegment[]>;
}

interface VideoMetadata {
  title: string;
  channelName: string;
  channelId: string;
  duration: number; // seconds
  publishDate: string;
  viewCount: number;
  description: string;
  tags: string[];
  categoryId: string;
  defaultLanguage?: string;
  chapters?: Array<{
    title: string;
    startTime: number;
    endTime: number;
  }>;
}

interface TranscriptOptions {
  language: 'auto' | 'en' | 'es' | 'fr' | 'de' | string;
  includeTimestamps: boolean;
  confidenceThreshold: number; // 0-1, minimum confidence for transcript segments
  speakerSeparation: boolean;
}

interface KeyMoment {
  startTime: number;
  endTime: number;
  content: string;
  importance: number; // 0-1 relevance score
  keywords: string[];
  momentType: 'concept_introduction' | 'example' | 'best_practice' | 'common_mistake' | 'summary';
}
```

##### Content Processing Pipeline
1. **Video Validation**: Check accessibility, duration limits (max 4 hours), and content type
2. **Metadata Extraction**: Get video details, chapters, and description
3. **Transcript Retrieval**: Extract captions with timestamp information
4. **Content Analysis**: Identify key moments, concepts, and learning objectives
5. **Segment Processing**: Extract specific timestamp ranges if requested
6. **Quality Assessment**: Evaluate transcript accuracy and content relevance

#### API Specification
```typescript
POST /api/content/extract-youtube
Authorization: Bearer <jwt_token>
Content-Type: application/json

Request Body:
{
  videoUrl: string; // YouTube URL or video ID
  processingOptions: {
    includeTranscript: boolean;
    extractKeyMoments: boolean;
    timestampSections?: Array<{
      start: number; // seconds
      end: number;   // seconds
      deck?: string; // optional description
    }>;
    transcriptLanguage?: string;
  };
  preferences?: {
    focusAreas?: string[]; // e.g., ["setup", "configuration", "examples"]
    difficultyLevel?: 'beginner' | 'intermediate' | 'advanced';
  };
}

Response:
{
  success: boolean;
  videoMetadata: {
    title: string;
    channelName: string;
    duration: number;
    publishDate: string;
    description: string;
    chapters?: Array<{title: string, startTime: number, endTime: number}>;
  };
  extractedContent: {
    fullTranscript?: string;
    keyMoments?: Array<{
      timestamp: number;
      content: string;
      importance: number;
      momentType: string;
    }>;
    processedSegments?: Array<{
      startTime: number;
      endTime: number;
      content: string;
      deck?: string;
    }>;
  };
  processingDetails: {
    transcriptAccuracy: number; // 0-1 confidence in transcript quality
    processingTime: number;
    contentQuality: number; // 0-1 educational value assessment
    languageDetected: string;
  };
}
```

#### YouTube-Specific Considerations
- **Copyright Compliance**: Respect YouTube's terms of service and fair use policies
- **Transcript Quality**: Handle auto-generated vs manual captions appropriately
- **Duration Limits**: Maximum 4-hour video processing to prevent excessive resource usage
- **Language Support**: Multi-language transcript processing with fallback options
- **Content Filtering**: Skip promotional content, ads, and off-topic segments

### 3. Hybrid Source Processing 🔗

#### Purpose
Combine multiple content sources (URL + YouTube + text input + files) to create comprehensive flashcard sets covering complex topics from multiple perspectives.

#### Use Cases
- **Complete Technology Learning**: Official docs + tutorial video + personal notes
- **Certification Preparation**: Study guide + practice videos + additional resources
- **Project Documentation**: README + walkthrough video + team knowledge
- **Concept Deep Dive**: Multiple expert explanations + examples + implementation details

#### Technical Implementation

##### Source Combination Engine
```typescript
interface HybridProcessingService {
  combineMultipleSources(sources: ContentSource[]): Promise<CombinedContent>;
  identifyContentOverlap(contents: ProcessedContent[]): Promise<OverlapAnalysis>;
  generateComprehensiveCards(combinedContent: CombinedContent): Promise<GeneratedCard[]>;
  calculateSourceContributions(sources: ContentSource[], cards: GeneratedCard[]): Promise<ContributionWeights>;
}

interface CombinedContent {
  unifiedContent: string;
  sourceMapping: Array<{
    content: string;
    sourceIndex: number;
    contributionWeight: number;
  }>;
  topicCoverage: {
    concepts: string[];
    examples: Array<{concept: string, sources: number[]}>;
    practicalApplications: string[];
    commonPatterns: string[];
  };
  redundancyAnalysis: {
    duplicateInformation: Array<{
      content: string;
      presentInSources: number[];
      confidenceLevel: number;
    }>;
    complementaryInformation: Array<{
      topic: string;
      sources: Array<{index: number, perspective: string}>;
    }>;
  };
}
```

##### Content Synthesis Process
1. **Source Processing**: Process each source independently
2. **Content Alignment**: Identify overlapping topics and concepts
3. **Redundancy Detection**: Find duplicate information across sources
4. **Gap Analysis**: Identify topics covered by some but not all sources
5. **Synthesis**: Combine unique insights while eliminating redundancy
6. **Attribution Mapping**: Track contribution weights for each source

#### API Specification
```typescript
POST /api/cards/generate-from-sources
Authorization: Bearer <jwt_token>
Content-Type: application/json

Request Body:
{
  sources: Array<{
    type: 'text_input' | 'file_upload' | 'url_extraction' | 'youtube_video';
    content?: string;
    url?: string;
    fileData?: string; // Base64 encoded
    deck?: string; // Optional deck for this specific source
  }>;
  processingOptions: {
    preferredCardCount: number;
    eliminateRedundancy: boolean;
    emphasizeComplementaryInfo: boolean;
    maintainSourceBalance: boolean; // Try to include content from all sources
  };
  preferences: {
    difficulty?: 'beginner' | 'intermediate' | 'advanced';
    contentType?: 'technical' | 'general' | 'certification';
    focusAreas?: string[];
  };
}

Response:
{
  success: boolean;
  generatedCards: Array<{
    id: string;
    question: string;
    answer: string;
    estimatedDifficulty: number;
    confidence: number;
    sources: Array<{
      type: ContentSourceType;
      url?: string;
      title?: string;
      contributionWeight: number; // 0-1, how much this source contributed
    }>;
    qualityMetrics: {
      clarity: number;
      uniqueness: number;
      technicalAccuracy: number;
      learningValue: number;
      sourceConsistency: number; // How well sources agree on this information
    };
    contentAnalysis: {
      primarySource: number; // Index of main contributing source
      crossValidated: boolean; // Information confirmed by multiple sources
      synthesized: boolean; // Content combined from multiple sources
    };
  }>;
  processingMetadata: {
    totalProcessingTime: number;
    sourcesProcessed: number;
    redundancyEliminated: number; // Percentage of duplicate content removed
    complementaryContentFound: number; // Unique insights from source combination
    overallSourceBalance: number; // 0-1, how evenly sources contributed
    aiModel: string;
    tokensUsed: number;
  };
}
```

## AI Quality Metrics System

### Quality Assessment Framework
```typescript
interface QualityMetrics {
  // Content Clarity (0-1)
  clarity: {
    score: number;
    factors: {
      questionClarity: number;   // How well-formed is the question
      answerCompleteness: number; // How complete is the answer
      languageSimplicity: number; // Appropriate complexity level
      structuralOrganization: number; // Logical flow and organization
    };
  };
  
  // Content Uniqueness (0-1)
  uniqueness: {
    score: number;
    factors: {
      noveltyLevel: number;      // How new/unique is this information
      commonKnowledgeAvoidance: number; // Avoids overly basic concepts
      specificityLevel: number;  // Specific vs generic information
      expertInsightLevel: number; // Contains expert-level insights
    };
  };
  
  // Technical Accuracy (0-1)
  technicalAccuracy: {
    score: number;
    factors: {
      factualCorrectness: number; // Verifiable technical facts
      currentRelevance: number;   // Up-to-date with current practices
      bestPracticeAlignment: number; // Follows established best practices
      crossSourceValidation: number; // Confirmed by multiple sources
    };
  };
  
  // Learning Value (0-1)
  learningValue: {
    score: number;
    factors: {
      practicalApplicability: number; // Can be applied in real work
      conceptualImportance: number;   // Fundamental concept understanding
      skillDevelopment: number;       // Contributes to skill building
      careerRelevance: number;        // Relevant to career progression
    };
  };
  
  // Overall Quality (computed weighted average)
  overallQuality: number;
}
```

### Quality Calculation Algorithm
```typescript
interface QualityCalculator {
  calculateQualityMetrics(
    cardContent: {question: string, answer: string},
    sources: ContentSource[],
    processingContext: ProcessingContext
  ): Promise<QualityMetrics>;
  
  validateQualityScores(metrics: QualityMetrics): QualityValidation;
  adjustQualityBasedOnFeedback(
    metrics: QualityMetrics, 
    userFeedback: CommunityRating[]
  ): QualityMetrics;
}

// Quality thresholds for different purposes
const QUALITY_THRESHOLDS = {
  communitySharing: {
    minimumOverallQuality: 0.7,
    minimumTechnicalAccuracy: 0.8,
    minimumClarity: 0.6
  },
  personalUse: {
    minimumOverallQuality: 0.5,
    minimumTechnicalAccuracy: 0.6,
    minimumClarity: 0.5
  },
  premiumRecommendations: {
    minimumOverallQuality: 0.85,
    minimumTechnicalAccuracy: 0.9,
    minimumClarity: 0.8
  }
};
```

## Source Attribution System

### Attribution Requirements
- **Full Transparency**: Every AI-generated card must include complete source information
- **Contribution Weighting**: Show relative contribution of each source (0-1 scale)
- **Human-Readable Attribution**: Generate descriptive attribution text
- **Link Preservation**: Maintain clickable links to original sources where appropriate

### Attribution Data Structure
```typescript
interface SourceAttribution {
  sources: Array<{
    type: ContentSourceType;
    url?: string;
    title: string;
    author?: string;
    publishDate?: string;
    domain?: string; // For URL sources
    channelName?: string; // For YouTube sources
    contributionWeight: number; // 0-1
  }>;
  
  // Human-readable attribution text
  attributionText: string; // e.g., "Generated from React Documentation (60%) and YouTube Tutorial by Dan Abramov (40%)"
  
  // Machine-readable attribution for API consumption
  attributionData: {
    primarySource: {
      type: ContentSourceType;
      identifier: string; // URL, video ID, etc.
      weight: number;
    };
    secondarySources?: Array<{
      type: ContentSourceType;
      identifier: string;
      weight: number;
    }>;
  };
}
```

### Attribution Generation
```typescript
interface AttributionGenerator {
  generateAttribution(
    sources: ProcessedContent[],
    contributionWeights: number[]
  ): SourceAttribution;
  
  formatForDisplay(attribution: SourceAttribution): string;
  formatForCommunitySharing(attribution: SourceAttribution): string;
  generateCitationFormat(attribution: SourceAttribution, format: 'APA' | 'MLA' | 'IEEE'): string;
}

// Example attribution generation
const generateAttributionText = (sources: SourceData[], weights: number[]): string => {
  const sortedSources = sources
    .map((source, index) => ({...source, weight: weights[index]}))
    .sort((a, b) => b.weight - a.weight);
    
  const primarySource = sortedSources[0];
  const secondarySources = sortedSources.slice(1).filter(s => s.weight > 0.1);
  
  let attribution = `Generated from ${primarySource.title}`;
  if (primarySource.weight < 1.0) {
    attribution += ` (${Math.round(primarySource.weight * 100)}%)`;
  }
  
  if (secondarySources.length > 0) {
    const secondaryText = secondarySources
      .map(s => `${s.title} (${Math.round(s.weight * 100)}%)`)
      .join(' and ');
    attribution += ` and ${secondaryText}`;
  }
  
  return attribution;
};
```

## Performance & Scalability Considerations

### Processing Limits
- **URL Content**: Maximum 15 seconds processing time per URL
- **YouTube Videos**: Maximum 30 seconds for videos under 60 minutes
- **Hybrid Processing**: Maximum 45 seconds total for up to 5 sources
- **Concurrent Requests**: Limit to 3 simultaneous processing requests per user

### Caching Strategy
```typescript
interface ContentCache {
  // Cache extracted content to avoid reprocessing same URLs
  urlContentCache: Map<string, {
    content: ExtractedContent;
    extractedAt: Date;
    expiresAt: Date; // 7 days for most content, 24 hours for news
  }>;
  
  // Cache YouTube transcripts and metadata
  youtubeCache: Map<string, {
    transcript: string;
    metadata: VideoMetadata;
    processedAt: Date;
    expiresAt: Date; // 30 days
  }>;
  
  // Cache AI quality assessments
  qualityMetricsCache: Map<string, {
    metrics: QualityMetrics;
    calculatedAt: Date;
    expiresAt: Date; // 14 days
  }>;
}
```

### Error Handling & Resilience
- **Graceful Degradation**: Continue processing available sources if some fail
- **Retry Logic**: Exponential backoff for transient failures
- **Fallback Processing**: Use alternative extraction methods when primary fails
- **User Feedback**: Clear error messages with suggested alternatives
- **Monitoring**: Track success rates and processing times for optimization

---

**Next Steps**: Continue with [Community Features](./community-features.md) for detailed collaboration and sharing specifications.