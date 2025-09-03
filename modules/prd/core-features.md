# Core Features Specification

Detailed specifications for all core features of the SuperMemo AI Card Generation System, building upon the existing SM-15 foundation.

## Feature Overview

### ✅ Existing Features (Ready to Use)
1. **User Authentication & Personal Learning** - Complete JWT-based system
2. **Advanced SM-15 Algorithm** - Sophisticated spaced repetition with OF Matrix
3. **Performance Analytics** - Comprehensive tracking via UserStatistic model
4. **Review System** - Core scheduling and grade processing

### 🔧 New Features (Development Required)
1. **Multi-Source AI Card Generation** - URL, YouTube, and hybrid content processing
2. **Manual Card Creation** - Simple individual card creation interface
3. **Enhanced Review Interface** - Auto-answer reveal and improved navigation
4. **Community Sharing System** - Deck-based sharing with privacy controls
5. **Quality Assurance** - AI metrics and community verification

---

## Feature 1: Multi-Source AI Card Generation 🤖

### Purpose
Transform any technical content (web pages, YouTube videos, text files, or combinations) into optimized flashcards using AI processing with quality metrics and source attribution.

### User Stories
- **As an engineer**, I want to convert a React documentation page into flashcards so I can memorize API patterns without manual card creation
- **As a team lead**, I want to process a tutorial video into key concept cards so my team can learn new technologies efficiently  
- **As a developer**, I want to combine multiple sources (docs + video + my notes) into comprehensive coverage of a complex topic

### Functional Requirements

#### Multi-Source Input Processing
```typescript
interface ContentSource {
  type: 'text_input' | 'file_upload' | 'url_extraction' | 'youtube_video' | 'hybrid_sources';
  content?: string;
  url?: string;
  fileData?: string;
  metadata?: {
    title?: string;
    author?: string;
    duration?: number;
    extractedAt: string;
    wordCount: number;
  };
}
```

#### URL Content Extraction
- **Supported Sites**: Documentation sites, GitHub, Stack Overflow, Medium, dev.to, personal blogs
- **Content Processing**: Extract main content, filter navigation and ads
- **Metadata Capture**: Title, author, publish date, domain
- **Error Handling**: Graceful degradation for restricted or dynamic content

#### YouTube Video Processing  
- **Transcript Extraction**: Automatic closed caption retrieval
- **Key Moment Identification**: AI-powered important timestamp detection
- **Segment Processing**: Extract specific video sections by timestamp
- **Metadata Capture**: Title, channel, duration, publish date

#### AI Quality Metrics
```typescript
interface QualityMetrics {
  clarity: number;           // 0-1: How clear and understandable
  uniqueness: number;        // 0-1: Originality vs existing content  
  technicalAccuracy: number; // 0-1: Factual correctness assessment
  learningValue: number;     // 0-1: Educational importance
  overallQuality: number;    // Computed weighted average
}
```

### Technical Implementation

#### API Endpoints
```typescript
// Multi-source card generation
POST /api/cards/generate-from-sources
Authorization: Bearer <jwt_token>
Content-Type: application/json

Request: {
  sources: ContentSource[];
  preferences: {
    preferredCardCount?: number;
    difficulty?: 'beginner' | 'intermediate' | 'advanced';
    contentType?: 'technical' | 'general' | 'certification';
    focusAreas?: string[];
  };
}

Response: {
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
      contributionWeight: number;
    }>;
    qualityMetrics: QualityMetrics;
  }>;
  processingMetadata: {
    totalProcessingTime: number;
    tokensUsed: number;
    sourcesProcessed: number;
    aiModel: string;
  };
}

// URL content extraction
POST /api/content/extract-url
Request: { url: string; extractionType: 'full' | 'summary' | 'key_points'; }
Response: { extractedContent: string; metadata: {...}; }

// YouTube video processing
POST /api/content/extract-youtube  
Request: { videoUrl: string; includeTranscript: boolean; timestampSections?: Array<{start: number, end: number}>; }
Response: { videoMetadata: {...}; transcript?: string; keyMoments?: Array<{...}>; }
```

#### Database Schema Extensions
```sql
-- Enhanced card tracking
ALTER TABLE cards ADD COLUMN content_sources JSONB DEFAULT '[]';
ALTER TABLE cards ADD COLUMN ai_quality_metrics JSONB DEFAULT '{}';
ALTER TABLE cards ADD COLUMN content_attribution TEXT;

-- Example data:
-- content_sources: [{"type": "url_extraction", "url": "https://...", "contributionWeight": 0.7}]
-- ai_quality_metrics: {"clarity": 0.9, "uniqueness": 0.8, "technicalAccuracy": 0.95, "learningValue": 0.85}
-- content_attribution: "Generated from React Official Documentation (70%) + Personal Notes (30%)"
```

### User Experience Flow
1. **Source Selection**: User chooses input method (URL, YouTube, text, file, or combination)
2. **Content Processing**: AI extracts and processes content with loading indicators
3. **Card Preview**: Generated cards displayed with quality metrics and source attribution
4. **Card Selection**: User selects desired cards using checkboxes (no editing allowed)
5. **Deck Assignment**: User assigns category/deck to selected cards
6. **Save & Schedule**: Cards saved to personal collection with SM-15 defaults

### Acceptance Criteria
- [ ] Successfully process and generate cards from web URLs
- [ ] Extract YouTube video transcripts and create relevant cards
- [ ] Handle hybrid sources (multiple inputs combined)
- [ ] Display quality metrics for all generated cards
- [ ] Show source attribution with contribution weights
- [ ] Graceful error handling for invalid URLs or processing failures
- [ ] Response time under 30 seconds for typical content processing

---

## Feature 2: Manual Card Creation ✏️

### Purpose
Provide a simple, fast interface for users to create individual flashcards manually when AI generation isn't needed or desired.

### User Stories
- **As an engineer**, I want to quickly create a card for a specific code snippet I want to remember
- **As a developer**, I want to add personal insights or clarifications to complement AI-generated content
- **As a team member**, I want to create cards for company-specific information not available in public sources

### Functional Requirements

#### Simple Card Creation Form
- **Question Field**: Multi-line text area (max 2000 characters)
- **Answer Field**: Multi-line text area (max 5000 characters)  
- **Deck Field**: Text input with autocomplete from user's existing decks
- **Preview Mode**: Show card appearance before saving
- **Immediate Save**: Single-click save with confirmation

#### Integration Features
- **Quick Review Option**: Start reviewing newly created card immediately
- **Batch Creation**: Save multiple cards in sequence without navigation
- **Template Support**: Common card formats for code snippets, definitions, procedures

### Technical Implementation

#### API Endpoint
```typescript
POST /api/cards/manual
Authorization: Bearer <jwt_token>
Content-Type: application/json

Request: {
  frontContent: string;
  backContent: string; 
  deck: string;
  sourceType: 'manual';
}

Response: {
  success: boolean;
  cardId: number;
  nextReviewDate: string;
  message: string;
}
```

#### User Interface Components
```typescript
interface ManualCardForm {
  question: string;
  answer: string;
  deck: string;
  isPreviewMode: boolean;
  availableDecks: string[];
}

// Form validation rules
const validation = {
  question: { required: true, maxLength: 2000 },
  answer: { required: true, maxLength: 5000 },
  deck: { required: true, pattern: /^[a-zA-Z0-9\s_-]+$/ }
};
```

### User Experience Flow
1. **Access Form**: Click "Create Card" → "Manual Card" option
2. **Fill Content**: Enter question and answer with real-time character count
3. **Assign Deck**: Type or select from existing decks with autocomplete
4. **Preview**: Optional preview of card appearance
5. **Save**: Single click save with success confirmation
6. **Next Action**: Option to create another card or start reviewing

### Acceptance Criteria
- [ ] Form validates input and prevents submission of empty fields
- [ ] Character limits enforced with visual indicators
- [ ] Deck autocomplete from user's existing decks
- [ ] Preview mode accurately reflects final card appearance
- [ ] Cards immediately available in review queue with SM-15 scheduling
- [ ] Form resets after successful submission for batch creation

---

## Feature 3: Enhanced Review Interface 📖

### Purpose
Improve the existing review system with auto-answer reveal, smooth navigation, and progress indicators while maintaining the sophisticated SM-15 algorithm.

### User Stories
- **As a user**, I want answers to appear automatically after grading so I don't need extra clicks
- **As a learner**, I want smooth transitions between cards so review sessions feel seamless
- **As an engineer**, I want to see my progress through the review session to maintain motivation

### Functional Requirements

#### Enhanced Review Flow
- **Auto-Answer Reveal**: Answer appears immediately after grade submission
- **Next Button Navigation**: Single click to advance to next card
- **Progress Indicators**: Cards remaining, session progress bar, estimated time
- **Session Management**: Pause/resume functionality, session statistics

#### Review Session Features
- **Smart Batching**: Group due cards into manageable review sessions (default 20 cards)
- **Deck Filtering**: Option to review specific categories or all due cards
- **Performance Feedback**: Real-time accuracy and timing statistics
- **Session Summary**: Review session results with key metrics

### Technical Implementation

#### Enhanced Review API
```typescript
GET /api/cards/due?deck={optional}&limit={20}
Authorization: Bearer <jwt_token>

Response: {
  dueCards: Array<{
    id: number;
    frontContent: string;
    backContent: string;
    aFactor: number;
    intervalDays: number;
    deck: string;
    lastReviewedAt?: string;
  }>;
  sessionMetadata: {
    totalDueCards: number;
    sessionSize: number;
    estimatedTime: number; // minutes
    availableDecks: string[];
  };
}

POST /api/reviews/grade
Authorization: Bearer <jwt_token>

Request: {
  cardId: number;
  grade: number; // 1-5
  responseTimeMs: number;
}

Response: {
  success: boolean;
  updatedCard: {
    nextReviewDate: string;
    newInterval: number;
    aFactorAfter: number;
  };
  nextCard?: {
    id: number;
    frontContent: string;
    // ... next card data
  };
  sessionProgress: {
    cardsCompleted: number;
    cardsRemaining: number;
    sessionAccuracy: number;
  };
}
```

#### Frontend State Management
```typescript
interface ReviewSession {
  cards: Card[];
  currentIndex: number;
  showAnswer: boolean;
  sessionStats: {
    cardsCompleted: number;
    totalCards: number;
    sessionAccuracy: number;
    averageResponseTime: number;
    startTime: Date;
  };
}
```

### User Experience Flow
1. **Start Session**: Dashboard shows due cards count, user clicks "Start Review"
2. **Card Presentation**: Question displayed with grade buttons hidden
3. **Answer Reveal**: User clicks "Show Answer", answer appears with grade buttons
4. **Grade Submission**: User selects grade (1-5), answer auto-appears, next button enabled
5. **Navigation**: Click "Next Card", smooth transition to next question
6. **Session Complete**: Summary statistics and return to dashboard

### Acceptance Criteria
- [ ] Answer appears automatically after grade submission without additional clicks
- [ ] Smooth transitions between cards with loading states
- [ ] Progress bar accurately shows session completion
- [ ] Deck filtering works with existing card data
- [ ] Session statistics update in real-time
- [ ] SM-15 algorithm calculations remain unchanged and accurate

---

## Feature 4: Community Sharing System 🤝

### Purpose
Enable selective knowledge sharing through deck-based privacy controls, peer rating, and quality assurance while maintaining personal learning privacy.

### User Stories
- **As a team lead**, I want to share my "React Best Practices" cards with my team but keep my "Personal Development" cards private
- **As an engineer**, I want to discover high-quality community cards about specific technologies I'm learning
- **As a contributor**, I want recognition for creating helpful content that other engineers use

### Functional Requirements

#### Deck-Based Privacy System
```typescript
enum PrivacyLevel {
  PRIVATE = 'private',        // Default: only owner can see
  TEAM = 'team',             // Visible to team members only
  COMMUNITY = 'community'    // Publicly available
}

interface DeckPrivacySettings {
  deck: string;
  privacyLevel: PrivacyLevel;
  cardCount: number;
  sharedCardCount: number;    // Cards actually shared within this deck
  communityViews: number;     // View statistics
  importCount: number;        // How many users imported these cards
}
```

#### Community Discovery & Rating
- **Browse by Deck**: Filter community cards by technical topics
- **Search Functionality**: Find cards by keywords in questions/answers
- **Quality Sorting**: Sort by rating, recency, import popularity
- **5-Star Rating System**: Rate cards with optional feedback
- **Source Attribution**: Display content origins and AI quality metrics

#### Card Import System
- **Selective Import**: Choose specific cards from community sets
- **Deck Assignment**: Assign imported cards to personal decks
- **SM-15 Integration**: Imported cards start with user's personal SM-15 parameters
- **Source Preservation**: Track original creator and import history

### Technical Implementation

#### Database Schema Extensions
```sql
-- Deck privacy management
CREATE TABLE deck_privacy_settings (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  deck VARCHAR(255) NOT NULL,
  privacy_level VARCHAR(20) DEFAULT 'private',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, deck)
);

-- Community ratings
CREATE TABLE community_ratings (
  id SERIAL PRIMARY KEY,
  card_id INTEGER REFERENCES cards(id) ON DELETE CASCADE,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  rating INTEGER CHECK (rating >= 1 AND rating <= 5),
  feedback TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(card_id, user_id)
);

-- Import tracking
CREATE TABLE community_card_imports (
  id SERIAL PRIMARY KEY,
  original_card_id INTEGER REFERENCES cards(id),
  imported_by_user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  imported_card_id INTEGER REFERENCES cards(id) ON DELETE CASCADE,
  imported_at TIMESTAMP DEFAULT NOW()
);

-- Enhanced cards table
ALTER TABLE cards ADD COLUMN is_community_shared BOOLEAN DEFAULT false;
ALTER TABLE cards ADD COLUMN community_rating DECIMAL(3,2) DEFAULT 0.0;
ALTER TABLE cards ADD COLUMN community_rating_count INTEGER DEFAULT 0;
```

#### Community API Endpoints
```typescript
// Browse community cards
GET /api/community/cards?deck={topic}&sort={rating|recent|popular}&limit={20}
Response: {
  communityCards: Array<{
    id: number;
    frontContent: string;
    backContent: string;
    deck: string;
    creatorName: string;
    averageRating: number;
    ratingCount: number;
    importCount: number;
    aiQualityMetrics?: QualityMetrics;
    contentSources: ContentSource[];
    contentAttribution: string;
    verified: boolean;
  }>;
  totalCount: number;
  availableDecks: string[];
}

// Rate community card
POST /api/community/cards/{cardId}/rate
Request: { rating: number; feedback?: string; }
Response: { success: boolean; updatedRating: number; }

// Import community card
POST /api/community/cards/{cardId}/import  
Request: { targetDeck: string; }
Response: { success: boolean; newCardId: number; }

// Manage deck privacy
PUT /api/decks/{deck}/privacy
Request: { privacyLevel: PrivacyLevel; }
Response: { success: boolean; affectedCards: number; }
```

### User Experience Flow

#### Sharing Workflow
1. **Deck Management**: User navigates to "Privacy Settings" page
2. **Privacy Selection**: Choose privacy level for each deck (private/team/community)
3. **Share Confirmation**: System shows card count and impact of sharing decision
4. **Publication**: Cards become available in community with attribution

#### Discovery Workflow  
1. **Community Browse**: User accesses "Community Cards" section
2. **Filter & Search**: Select topic decks, search keywords, sort by quality
3. **Card Preview**: View cards with ratings, source attribution, and quality metrics
4. **Import Selection**: Choose cards to import and assign to personal decks
5. **Integration**: Imported cards appear in personal review queue with SM-15 scheduling

### Acceptance Criteria
- [ ] Deck privacy settings control card visibility accurately
- [ ] Community cards display with all quality metrics and attribution
- [ ] Rating system updates averages correctly and prevents duplicate ratings
- [ ] Import process creates new cards in user's collection with preserved sources
- [ ] Search and filtering work across all community content
- [ ] Privacy changes take effect immediately across all community features

---

## Integration Requirements

### SM-15 Algorithm Compatibility
- All new features must preserve existing SM-15 calculations
- Imported cards start with default SM-15 parameters (A-Factor: 2.5, interval: 1 day)
- Review history and performance analytics continue functioning
- Quality metrics do not influence SM-15 scheduling decisions

### Performance Requirements
- URL content extraction: < 15 seconds for typical web pages
- YouTube processing: < 30 seconds for videos under 60 minutes
- Community card browsing: < 2 seconds for filtered results
- Card import process: < 5 seconds for batch imports up to 50 cards

### Data Privacy & Security
- All personal cards remain private by default
- Community sharing requires explicit user consent
- Source attribution preserves original creator recognition
- User can revoke community sharing at any time
- No personal performance data shared in community features

---

**Next Steps**: Review [Multi-Source Content](./multi-source-content.md) for detailed technical specifications of URL and YouTube processing capabilities.