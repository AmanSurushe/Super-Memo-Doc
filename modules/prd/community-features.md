# Community Features Specification

Detailed specifications for collaborative learning features including label-based sharing, peer rating systems, content verification, and contributor recognition while maintaining privacy and quality standards.

## Overview

The community features transform individual learning into collaborative team knowledge building while preserving the scientific rigor of the SM-15 spaced repetition system. Users maintain full control over their privacy while enabling selective knowledge sharing with quality assurance mechanisms.

## Core Principles

### Privacy-First Design
- **Default Private**: All cards and labels are private by default
- **Explicit Consent**: Sharing requires deliberate user action
- **Granular Control**: Users control sharing at the label level with individual card exclusions
- **Revocable Sharing**: Users can make shared content private at any time

### Quality Assurance
- **AI Quality Metrics**: Automated quality assessment before sharing
- **Peer Review**: Community rating and feedback system
- **Content Verification**: Accuracy checking by subject matter experts
- **Attribution Transparency**: Full source attribution for all shared content

### Collaborative Learning
- **Knowledge Discovery**: Browse and search community content by topics
- **Selective Import**: Choose specific cards to add to personal collections
- **Contributor Recognition**: Acknowledge valuable content creators
- **Team Building**: Foster collaborative learning culture within organizations

---

## Feature 1: Label-Based Privacy System 🔒

### Purpose
Enable users to selectively share entire categories of cards while maintaining privacy for personal content, with granular control over individual cards within shared labels.

### Privacy Levels
```typescript
enum PrivacyLevel {
  PRIVATE = 'private',        // Only owner can see (default)
  TEAM = 'team',             // Visible to organization/team members only
  COMMUNITY = 'community'    // Publicly available to all users
}

interface LabelPrivacySettings {
  userId: number;
  label: string;
  privacyLevel: PrivacyLevel;
  cardCount: number;          // Total cards in this label
  sharedCardCount: number;    // Cards actually shared (excludes individually private cards)
  excludedCardIds: number[];  // Individual cards excluded from sharing
  createdAt: Date;
  updatedAt: Date;
}
```

### User Experience Flow

#### Setting Label Privacy
1. **Privacy Dashboard**: User navigates to "Label Privacy Settings"
2. **Label Overview**: Display all labels with current privacy levels and card counts
3. **Privacy Selection**: Click to change privacy level (Private → Team → Community)
4. **Impact Preview**: Show how many cards will be affected by privacy change
5. **Confirmation**: Confirm sharing decision with clear impact statement
6. **Individual Exclusions**: Option to exclude specific cards from shared labels

#### Privacy Management Interface
```typescript
interface PrivacyManagementUI {
  labelSummary: Array<{
    label: string;
    cardCount: number;
    privacyLevel: PrivacyLevel;
    sharedCount: number;
    excludedCount: number;
    communityStats?: {
      viewCount: number;
      importCount: number;
      averageRating: number;
    };
  }>;
  
  bulkOperations: {
    setMultipleLabelsPrivate: (labels: string[]) => void;
    setMultipleLabelsPublic: (labels: string[]) => void;
    excludeCardsFromSharing: (cardIds: number[]) => void;
  };
  
  privacyAnalytics: {
    totalSharedCards: number;
    totalPrivateCards: number;
    communityImpact: {
      totalViews: number;
      totalImports: number;
      averageRating: number;
    };
  };
}
```

### Technical Implementation

#### Database Schema
```sql
-- Label privacy settings
CREATE TABLE label_privacy_settings (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  label VARCHAR(255) NOT NULL,
  privacy_level VARCHAR(20) DEFAULT 'private',
  excluded_card_ids INTEGER[] DEFAULT '{}', -- Individual cards excluded from sharing
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, label)
);

-- Enhanced cards table for sharing
ALTER TABLE cards ADD COLUMN is_community_shared BOOLEAN DEFAULT false;
ALTER TABLE cards ADD COLUMN shared_at TIMESTAMP;
ALTER TABLE cards ADD COLUMN community_views INTEGER DEFAULT 0;

-- Indexes for performance
CREATE INDEX idx_label_privacy_user_level ON label_privacy_settings(user_id, privacy_level);
CREATE INDEX idx_cards_community_shared ON cards(is_community_shared, label) WHERE is_community_shared = true;
```

#### API Endpoints
```typescript
// Get user's privacy settings
GET /api/privacy/labels
Authorization: Bearer <jwt_token>

Response: {
  labelSettings: Array<{
    label: string;
    privacyLevel: PrivacyLevel;
    cardCount: number;
    sharedCardCount: number;
    excludedCardIds: number[];
    communityStats?: {
      views: number;
      imports: number;
      averageRating: number;
    };
  }>;
  summary: {
    totalLabels: number;
    privateLabels: number;
    teamLabels: number;
    communityLabels: number;
    totalSharedCards: number;
  };
}

// Update label privacy level
PUT /api/privacy/labels/{label}
Authorization: Bearer <jwt_token>
Content-Type: application/json

Request: {
  privacyLevel: PrivacyLevel;
  excludedCardIds?: number[]; // Optional: specific cards to exclude from sharing
}

Response: {
  success: boolean;
  affectedCards: number;
  newPrivacyLevel: PrivacyLevel;
  communityImpact?: {
    newSharedCards: number;
    removedSharedCards: number;
  };
}

// Bulk privacy operations
POST /api/privacy/labels/bulk-update
Authorization: Bearer <jwt_token>

Request: {
  operations: Array<{
    label: string;
    privacyLevel: PrivacyLevel;
    excludedCardIds?: number[];
  }>;
}

Response: {
  success: boolean;
  results: Array<{
    label: string;
    success: boolean;
    affectedCards: number;
    error?: string;
  }>;
}
```

### Privacy Enforcement
- **Real-time Updates**: Privacy changes take effect immediately across all community features
- **Access Control**: API endpoints validate privacy levels before returning content
- **Audit Trail**: Track privacy changes for security and user transparency
- **Data Cleanup**: Automatically remove community statistics when content becomes private

---

## Feature 2: Community Rating & Feedback System ⭐

### Purpose
Enable community members to rate shared cards for quality, accuracy, and usefulness while providing constructive feedback for content improvement.

### Rating System Design

#### 5-Star Rating Scale
```typescript
interface CommunityRating {
  cardId: number;
  userId: number;
  rating: number; // 1-5 stars
  feedback?: string; // Optional constructive feedback
  ratingCategories: {
    accuracy: number; // 1-5: How factually accurate
    clarity: number;  // 1-5: How clear and understandable
    usefulness: number; // 1-5: How practically useful
    difficulty: number; // 1-5: Difficulty level assessment
  };
  tags: string[]; // e.g., ["beginner-friendly", "advanced", "practical"]
  createdAt: Date;
  updatedAt: Date;
}

interface AggregatedRating {
  cardId: number;
  averageRating: number;
  totalRatings: number;
  ratingDistribution: {
    fiveStars: number;
    fourStars: number;
    threeStars: number;
    twoStars: number;
    oneStar: number;
  };
  categoryAverages: {
    accuracy: number;
    clarity: number;
    usefulness: number;
    difficulty: number;
  };
  commonTags: Array<{tag: string, count: number}>;
}
```

#### Rating Quality Controls
- **One Rating Per User**: Prevent duplicate ratings from same user
- **Rating History**: Track rating changes and prevent gaming
- **Minimum Interaction**: Require viewing card before rating
- **Constructive Feedback**: Encourage helpful, specific feedback
- **Moderation**: Flag inappropriate or unhelpful ratings

### User Experience Flow

#### Rating Workflow
1. **Card Discovery**: User browses community cards by topic or search
2. **Card Review**: User views card content, source attribution, and existing ratings
3. **Rating Submission**: User provides 1-5 star rating with optional detailed feedback
4. **Category Rating**: Optional detailed ratings for accuracy, clarity, usefulness, difficulty
5. **Tag Addition**: Optional tags to help other users find relevant content
6. **Confirmation**: Rating submitted with thank you message and impact notification

#### Rating Display
```typescript
interface RatingDisplayComponent {
  overallRating: {
    averageStars: number; // 0.0 - 5.0
    totalRatings: number;
    ratingDistribution: number[]; // [1-star count, 2-star count, etc.]
  };
  
  categoryBreakdown: {
    accuracy: number;
    clarity: number;
    usefulness: number;
    difficulty: number;
  };
  
  recentFeedback: Array<{
    rating: number;
    feedback: string;
    username: string; // Anonymous option available
    createdAt: Date;
    helpfulVotes: number; // Other users found this feedback helpful
  }>;
  
  tags: Array<{
    tag: string;
    count: number;
    trending: boolean;
  }>;
}
```

### Technical Implementation

#### Database Schema
```sql
-- Community ratings
CREATE TABLE community_ratings (
  id SERIAL PRIMARY KEY,
  card_id INTEGER REFERENCES cards(id) ON DELETE CASCADE,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  rating INTEGER CHECK (rating >= 1 AND rating <= 5),
  accuracy_rating INTEGER CHECK (accuracy_rating >= 1 AND accuracy_rating <= 5),
  clarity_rating INTEGER CHECK (clarity_rating >= 1 AND clarity_rating <= 5),
  usefulness_rating INTEGER CHECK (usefulness_rating >= 1 AND usefulness_rating <= 5),
  difficulty_rating INTEGER CHECK (difficulty_rating >= 1 AND difficulty_rating <= 5),
  feedback TEXT,
  tags TEXT[] DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(card_id, user_id)
);

-- Aggregated rating cache for performance
CREATE TABLE community_rating_cache (
  card_id INTEGER PRIMARY KEY REFERENCES cards(id) ON DELETE CASCADE,
  average_rating DECIMAL(3,2) DEFAULT 0.0,
  total_ratings INTEGER DEFAULT 0,
  rating_distribution JSONB DEFAULT '{"1":0,"2":0,"3":0,"4":0,"5":0}',
  category_averages JSONB DEFAULT '{}',
  common_tags JSONB DEFAULT '[]',
  last_updated TIMESTAMP DEFAULT NOW()
);

-- Rating helpfulness votes
CREATE TABLE rating_helpfulness (
  id SERIAL PRIMARY KEY,
  rating_id INTEGER REFERENCES community_ratings(id) ON DELETE CASCADE,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  is_helpful BOOLEAN,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(rating_id, user_id)
);
```

#### API Endpoints
```typescript
// Submit card rating
POST /api/community/cards/{cardId}/rate
Authorization: Bearer <jwt_token>

Request: {
  rating: number; // 1-5 overall rating
  categoryRatings?: {
    accuracy: number;
    clarity: number;
    usefulness: number;
    difficulty: number;
  };
  feedback?: string;
  tags?: string[];
  anonymous?: boolean; // Hide username in public feedback
}

Response: {
  success: boolean;
  updatedAverageRating: number;
  totalRatings: number;
  yourPreviousRating?: number; // If this was an update
}

// Get card ratings and feedback
GET /api/community/cards/{cardId}/ratings
Response: {
  aggregatedRating: AggregatedRating;
  recentFeedback: Array<{
    rating: number;
    feedback: string;
    username: string;
    anonymous: boolean;
    createdAt: Date;
    helpfulVotes: number;
  }>;
  userRating?: number; // Current user's rating if any
}

// Vote on rating helpfulness
POST /api/community/ratings/{ratingId}/helpful
Request: { isHelpful: boolean; }
Response: { success: boolean; newHelpfulCount: number; }
```

### Rating Analytics & Insights
```typescript
interface RatingAnalytics {
  // For card creators
  creatorInsights: {
    averageRatingAcrossAllCards: number;
    totalRatingsReceived: number;
    ratingTrends: Array<{date: Date, averageRating: number}>;
    commonFeedbackThemes: Array<{theme: string, frequency: number}>;
    improvementSuggestions: string[];
  };
  
  // For the community
  communityInsights: {
    highestRatedCards: Array<{cardId: number, rating: number, label: string}>;
    mostRatedCards: Array<{cardId: number, ratingCount: number}>;
    trendingTags: Array<{tag: string, growth: number}>;
    qualityTrends: {
      overallQualityImprovement: number;
      categoryTrends: {[category: string]: number};
    };
  };
}
```

---

## Feature 3: Content Discovery & Search 🔍

### Purpose
Enable users to efficiently discover high-quality community content through powerful search, filtering, and recommendation systems.

### Discovery Features

#### Advanced Search & Filtering
```typescript
interface CommunitySearchFilters {
  // Content filters
  query?: string; // Full-text search across questions and answers
  labels?: string[]; // Filter by specific topics/categories
  sourceTypes?: ContentSourceType[]; // Filter by how cards were created
  
  // Quality filters
  minimumRating?: number; // Only show cards above rating threshold
  minimumRatings?: number; // Only show cards with sufficient rating volume
  verifiedOnly?: boolean; // Only community-verified content
  aiQualityThreshold?: number; // Minimum AI quality score
  
  // Temporal filters
  dateRange?: {
    from: Date;
    to: Date;
  };
  recentlyUpdated?: boolean; // Content updated in last 30 days
  
  // Popularity filters
  trending?: boolean; // Recently gaining popularity
  mostImported?: boolean; // Frequently imported by users
  topRated?: boolean; // Highest rated content
  
  // Difficulty filters
  difficultyLevel?: 'beginner' | 'intermediate' | 'advanced';
  estimatedDifficulty?: {
    min: number; // Minimum A-Factor
    max: number; // Maximum A-Factor
  };
}

interface SearchResults {
  cards: Array<{
    id: number;
    frontContent: string;
    backContent: string;
    label: string;
    creator: {
      username: string;
      verified: boolean;
      reputation: number;
    };
    ratings: {
      average: number;
      count: number;
      distribution: number[];
    };
    qualityMetrics: {
      aiQualityScore: number;
      communityVerified: boolean;
      sourceAttribution: string;
    };
    popularity: {
      viewCount: number;
      importCount: number;
      bookmarkCount: number;
    };
    metadata: {
      createdAt: Date;
      lastUpdated: Date;
      sourceTypes: ContentSourceType[];
      tags: string[];
    };
  }>;
  
  aggregations: {
    totalResults: number;
    availableLabels: Array<{label: string, count: number}>;
    ratingDistribution: {[rating: string]: number};
    popularTags: Array<{tag: string, count: number}>;
    sourceTypeDistribution: {[type: string]: number};
  };
  
  suggestions: {
    relatedQueries: string[];
    recommendedLabels: string[];
    similarUsers: Array<{username: string, sharedInterests: string[]}>;
  };
}
```

#### Smart Recommendations
```typescript
interface RecommendationEngine {
  // Personalized recommendations based on user's learning history
  personalizedRecommendations: (userId: number) => Promise<RecommendationSet>;
  
  // Similar content recommendations
  findSimilarCards: (cardId: number, limit: number) => Promise<Card[]>;
  
  // Trending content detection
  identifyTrendingContent: (timeframe: 'day' | 'week' | 'month') => Promise<TrendingContent>;
  
  // Quality-based recommendations
  getHighQualityNewContent: (userId: number) => Promise<Card[]>;
}

interface RecommendationSet {
  forYou: Card[]; // Based on user's labels and activity
  trending: Card[]; // Currently popular content
  similarUsers: Card[]; // What similar users are studying
  newHighQuality: Card[]; // Recently added high-quality content
  complementary: Card[]; // Content that complements user's existing cards
}
```

### Technical Implementation

#### Search API
```typescript
GET /api/community/search
Authorization: Bearer <jwt_token>

Query Parameters: {
  q?: string; // Search query
  labels?: string; // Comma-separated list
  minRating?: number;
  sourceTypes?: string; // Comma-separated list
  sort?: 'relevance' | 'rating' | 'recent' | 'popular' | 'trending';
  limit?: number; // Default 20, max 100
  offset?: number; // For pagination
  verified?: boolean;
  difficulty?: string;
}

Response: SearchResults;

// Advanced search endpoint
POST /api/community/search/advanced
Authorization: Bearer <jwt_token>

Request: {
  filters: CommunitySearchFilters;
  sorting: Array<{
    field: 'rating' | 'popularity' | 'quality' | 'recency';
    direction: 'asc' | 'desc';
    weight: number; // For multi-criteria sorting
  }>;
  personalization: {
    includePersonalized: boolean;
    userPreferences?: {
      preferredDifficulty: string;
      interestAreas: string[];
      excludeLabels: string[];
    };
  };
}

Response: SearchResults & {
  personalizedResults?: Card[];
  recommendations?: RecommendationSet;
}
```

#### Search Performance Optimization
```sql
-- Full-text search indexes
CREATE INDEX idx_cards_fulltext ON cards USING GIN(to_tsvector('english', front_content || ' ' || back_content));
CREATE INDEX idx_cards_label_rating ON cards(label, community_rating) WHERE is_community_shared = true;
CREATE INDEX idx_cards_quality_rating ON cards(ai_quality_metrics, community_rating) WHERE is_community_shared = true;

-- Materialized view for trending content
CREATE MATERIALIZED VIEW trending_cards AS
SELECT 
  card_id,
  label,
  community_rating,
  community_views,
  import_count,
  (community_views * 0.3 + import_count * 0.7 + community_rating * 0.5) as trend_score,
  created_at
FROM cards 
WHERE is_community_shared = true 
  AND created_at >= NOW() - INTERVAL '30 days'
ORDER BY trend_score DESC;

-- Refresh trending view daily
CREATE OR REPLACE FUNCTION refresh_trending_cards()
RETURNS void AS $$
BEGIN
  REFRESH MATERIALIZED VIEW CONCURRENTLY trending_cards;
END;
$$ LANGUAGE plpgsql;
```

---

## Feature 4: Card Import & Personal Integration 📥

### Purpose
Enable users to selectively import community cards into their personal collections while preserving SM-15 personalization and maintaining source attribution.

### Import Process Design

#### Selective Import Interface
```typescript
interface ImportSelection {
  cards: Array<{
    communityCardId: number;
    selected: boolean;
    targetLabel: string; // User's personal label for imported card
    customizations?: {
      addPersonalNotes?: string; // Additional context for personal use
      adjustDifficulty?: number; // Override estimated difficulty
      priority?: 'high' | 'medium' | 'low'; // Personal priority level
    };
  }>;
  
  importSettings: {
    preserveOriginalLabels: boolean;
    createNewLabelsIfNeeded: boolean;
    notifyOriginalCreator: boolean; // Thank the creator for useful content
    addToReviewQueue: boolean; // Start reviewing immediately
  };
}

interface ImportResult {
  successfulImports: Array<{
    originalCardId: number;
    newCardId: number;
    assignedLabel: string;
    nextReviewDate: Date;
  }>;
  
  failedImports: Array<{
    originalCardId: number;
    error: string;
    suggestion?: string;
  }>;
  
  summary: {
    totalAttempted: number;
    successful: number;
    failed: number;
    newLabelsCreated: string[];
    addedToReviewQueue: number;
  };
}
```

#### SM-15 Integration for Imported Cards
```typescript
interface ImportedCardInitialization {
  // SM-15 default parameters for new imports
  initialParameters: {
    aFactor: 2.5; // Default difficulty
    repetitionCount: 0;
    intervalDays: 1;
    lapsesCount: 0;
    nextReviewDate: Date; // Tomorrow by default
  };
  
  // Preserve source information
  sourcePreservation: {
    originalCreatorId: number;
    importedAt: Date;
    originalCardId: number;
    sourceAttribution: string; // Preserved from original
    communityRatingAtImport: number;
  };
  
  // Personal customizations
  personalizations: {
    personalNotes?: string;
    customLabel: string;
    userDifficultyOverride?: number;
    importReason?: string; // Why user imported this card
  };
}
```

### Technical Implementation

#### Database Schema Extensions
```sql
-- Track imported cards
CREATE TABLE community_card_imports (
  id SERIAL PRIMARY KEY,
  original_card_id INTEGER REFERENCES cards(id) ON DELETE SET NULL, -- Original may be deleted
  imported_by_user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  imported_card_id INTEGER REFERENCES cards(id) ON DELETE CASCADE, -- User's copy
  imported_at TIMESTAMP DEFAULT NOW(),
  import_reason TEXT,
  original_creator_notified BOOLEAN DEFAULT false
);

-- Enhanced cards table for import tracking
ALTER TABLE cards ADD COLUMN imported_from_card_id INTEGER REFERENCES cards(id) ON DELETE SET NULL;
ALTER TABLE cards ADD COLUMN import_metadata JSONB DEFAULT '{}';

-- Example import_metadata:
-- {
--   "originalCreator": "john_doe", 
--   "importedAt": "2024-01-15T10:30:00Z",
--   "communityRatingAtImport": 4.2,
--   "personalNotes": "Focus on practical examples",
--   "importReason": "Needed for React certification prep"
-- }
```

#### Import API Endpoints
```typescript
// Import selected community cards
POST /api/community/cards/import
Authorization: Bearer <jwt_token>

Request: {
  imports: Array<{
    cardId: number;
    targetLabel: string;
    personalNotes?: string;
    customDifficulty?: number;
    importReason?: string;
  }>;
  settings: {
    notifyCreators: boolean;
    addToReviewQueue: boolean;
    preserveSourceAttribution: boolean;
  };
}

Response: ImportResult;

// Get import history
GET /api/user/imports
Authorization: Bearer <jwt_token>

Response: {
  recentImports: Array<{
    importedAt: Date;
    originalCard: {
      id: number;
      frontContent: string;
      creator: string;
      label: string;
    };
    yourCard: {
      id: number;
      label: string;
      nextReviewDate: Date;
      currentInterval: number;
    };
    performance: {
      reviewsCompleted: number;
      averageGrade: number;
      currentStreak: number;
    };
  }>;
  
  importStatistics: {
    totalImported: number;
    activelyReviewing: number;
    mastered: number; // Interval > 30 days
    struggling: number; // Recent lapses
    favoriteCreators: Array<{creator: string, importCount: number}>;
  };
}

// Track import impact for creators
GET /api/community/my-content/impact
Authorization: Bearer <jwt_token>

Response: {
  totalImports: number;
  recentImports: Array<{
    cardId: number;
    frontContent: string;
    importedBy: string; // Anonymous option
    importedAt: Date;
    importReason?: string;
  }>;
  
  impactMetrics: {
    totalReach: number; // Unique users who imported content
    helpfulnessScore: number; // Based on import frequency and ratings
    topPerformingCards: Array<{
      cardId: number;
      importCount: number;
      averageRating: number;
      label: string;
    }>;
  };
}
```

### Import Quality Assurance
- **Duplicate Detection**: Prevent importing cards similar to existing personal cards
- **Attribution Preservation**: Maintain source attribution through import process
- **Creator Notification**: Optional thank-you notifications to original creators
- **Import Analytics**: Track which community content provides most value

---

## Feature 5: Contributor Recognition & Community Building 🏆

### Purpose
Foster a collaborative learning community by recognizing valuable contributors, encouraging quality content creation, and building reputation systems that promote knowledge sharing.

### Recognition Systems

#### Contributor Badges & Achievements
```typescript
interface ContributorBadge {
  id: string;
  name: string;
  description: string;
  icon: string;
  criteria: {
    type: 'threshold' | 'quality' | 'impact' | 'consistency';
    requirements: {
      minSharedCards?: number;
      minAverageRating?: number;
      minImports?: number;
      streakDays?: number;
      qualityThreshold?: number;
    };
  };
  rarity: 'common' | 'rare' | 'epic' | 'legendary';
  earnedBy?: {
    userId: number;
    earnedAt: Date;
    progress?: number; // 0-1 for partially earned badges
  };
}

const COMMUNITY_BADGES: ContributorBadge[] = [
  {
    id: 'first_share',
    name: 'First Share',
    description: 'Shared your first community card',
    criteria: { type: 'threshold', requirements: { minSharedCards: 1 }},
    rarity: 'common'
  },
  {
    id: 'quality_creator',
    name: 'Quality Creator',
    description: 'Maintained 4.5+ average rating across 25+ shared cards',
    criteria: { type: 'quality', requirements: { minSharedCards: 25, minAverageRating: 4.5 }},
    rarity: 'epic'
  },
  {
    id: 'team_favorite',
    name: 'Team Favorite',
    description: 'Your cards have been imported 100+ times',
    criteria: { type: 'impact', requirements: { minImports: 100 }},
    rarity: 'rare'
  },
  {
    id: 'knowledge_guru',
    name: 'Knowledge Guru',
    description: 'Legendary contributor with 500+ quality imports',
    criteria: { type: 'impact', requirements: { minImports: 500, minAverageRating: 4.0 }},
    rarity: 'legendary'
  }
];
```

#### Reputation System
```typescript
interface UserReputation {
  userId: number;
  totalScore: number; // Weighted score based on all activities
  
  contributionMetrics: {
    cardsShared: number;
    averageRating: number;
    totalImports: number;
    totalViews: number;
    verifiedCards: number; // Community-verified for accuracy
  };
  
  qualityMetrics: {
    averageAIQuality: number;
    consistentQuality: boolean; // Maintains quality over time
    expertiseAreas: Array<{
      label: string;
      cardCount: number;
      averageRating: number;
      recognizedExpert: boolean; // Community recognition
    }>;
  };
  
  communityEngagement: {
    ratingsGiven: number;
    helpfulFeedbackCount: number; // Feedback marked as helpful by others
    mentorshipActivity: number; // Helping new contributors
  };
  
  badges: ContributorBadge[];
  level: number; // 1-100 based on total score
  nextLevelProgress: number; // 0-1
}
```

### Community Recognition Features

#### Contributor Leaderboards
```typescript
interface CommunityLeaderboards {
  topContributors: Array<{
    rank: number;
    username: string;
    reputation: number;
    totalSharedCards: number;
    averageRating: number;
    specialties: string[]; // Top expertise areas
    badges: string[]; // Badge IDs
  }>;
  
  risingStars: Array<{
    username: string;
    recentShares: number;
    growthRate: number; // Reputation growth in last 30 days
    potentialBadges: string[]; // Badges close to earning
  }>;
  
  qualityLeaders: Array<{
    username: string;
    averageRating: number;
    minCardCount: number; // Must have minimum volume for quality ranking
    consistencyScore: number; // How consistent their quality is
  }>;
  
  categoryExperts: {
    [category: string]: Array<{
      username: string;
      categoryScore: number;
      cardsInCategory: number;
      averageRating: number;
    }>;
  };
}
```

#### Thank You System
```typescript
interface ThankYouSystem {
  // Users can thank creators for helpful content
  sendThankYou: (fromUserId: number, toUserId: number, cardId: number, message?: string) => void;
  
  // Aggregate thank you messages for creators
  getThankYouSummary: (userId: number) => {
    totalReceived: number;
    recentMessages: Array<{
      fromUser: string;
      cardId: number;
      message?: string;
      receivedAt: Date;
    }>;
    impactSummary: {
      peopleHelped: number;
      topHelpfulCards: Array<{cardId: number, thankYouCount: number}>;
      encouragingMessages: string[];
    };
  };
}
```

### Technical Implementation

#### Database Schema
```sql
-- User reputation and metrics
CREATE TABLE user_reputation (
  user_id INTEGER PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  total_score INTEGER DEFAULT 0,
  cards_shared INTEGER DEFAULT 0,
  average_rating DECIMAL(3,2) DEFAULT 0.0,
  total_imports INTEGER DEFAULT 0,
  total_views INTEGER DEFAULT 0,
  verified_cards INTEGER DEFAULT 0,
  ratings_given INTEGER DEFAULT 0,
  helpful_feedback_count INTEGER DEFAULT 0,
  level INTEGER DEFAULT 1,
  badges JSONB DEFAULT '[]', -- Array of badge IDs
  expertise_areas JSONB DEFAULT '{}', -- {label: {cardCount, avgRating, expert}}
  last_updated TIMESTAMP DEFAULT NOW()
);

-- Badge achievements tracking
CREATE TABLE badge_achievements (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  badge_id VARCHAR(50) NOT NULL,
  earned_at TIMESTAMP DEFAULT NOW(),
  progress DECIMAL(3,2) DEFAULT 1.0, -- Progress towards earning (1.0 = earned)
  UNIQUE(user_id, badge_id)
);

-- Thank you messages
CREATE TABLE community_thank_you (
  id SERIAL PRIMARY KEY,
  from_user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  to_user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  card_id INTEGER REFERENCES cards(id) ON DELETE CASCADE,
  message TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Leaderboard cache (updated daily)
CREATE TABLE community_leaderboards (
  id SERIAL PRIMARY KEY,
  leaderboard_type VARCHAR(50) NOT NULL, -- 'contributors', 'quality', 'rising_stars'
  category VARCHAR(100), -- Optional category for specialized leaderboards
  rankings JSONB NOT NULL, -- Array of ranked users with metrics
  generated_at TIMESTAMP DEFAULT NOW()
);
```

#### Recognition API Endpoints
```typescript
// Get user's reputation and achievements
GET /api/community/reputation/{userId}
Response: UserReputation;

// Get community leaderboards
GET /api/community/leaderboards?type=contributors&category=react&limit=50
Response: CommunityLeaderboards;

// Send thank you message
POST /api/community/thank-you
Request: {
  recipientUserId: number;
  cardId: number;
  message?: string;
  anonymous?: boolean;
}
Response: { success: boolean; }

// Get badge progress
GET /api/community/badges/progress
Authorization: Bearer <jwt_token>
Response: {
  earnedBadges: ContributorBadge[];
  inProgress: Array<{
    badge: ContributorBadge;
    currentProgress: number;
    nextMilestone: string;
    estimatedTimeToComplete: string;
  }>;
  available: ContributorBadge[]; // Not yet started
}
```

### Community Moderation & Quality Control

#### Content Moderation System
```typescript
interface ContentModerationSystem {
  // Automated quality checks
  automaticQualityAssessment: {
    aiQualityThreshold: number; // Minimum AI quality for sharing
    ratingThreshold: number; // Minimum community rating to remain shared
    reportThreshold: number; // Number of reports before review
  };
  
  // Community moderation features
  reportingSystem: {
    reportCard: (cardId: number, reason: string, details: string) => void;
    reportUser: (userId: number, reason: string, evidence: string) => void;
    moderatorReview: (reportId: number, action: 'approve' | 'hide' | 'delete') => void;
  };
  
  // Quality enforcement
  qualityEnforcement: {
    hideCards: number[]; // Cards hidden from community for quality issues
    shadowBannedUsers: number[]; // Users whose new content requires pre-approval
    verifiedContributors: number[]; // Users whose content is pre-approved
  };
}
```

#### Community Guidelines & Standards
- **Quality Standards**: Minimum AI quality scores for sharing
- **Attribution Requirements**: Proper source citation for all shared content
- **Respectful Communication**: Professional and constructive feedback only
- **Educational Focus**: Content must have clear learning value
- **Accuracy Verification**: Community verification system for technical content

---

**Next Steps**: Continue with [API Specifications](./api-specifications.md) for complete technical implementation details.