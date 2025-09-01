# Constants and Formulas Reference

Complete reference of all constants, formulas, and configuration parameters used in the SM-15 algorithm.

## 🔢 Core Algorithm Constants

### Grade Scale
```typescript
const GRADE_SCALE = {
  MINIMUM: 1,
  MAXIMUM: 5,
  FAILED: 1,
  HARD: 2,
  GOOD: 3,
  EASY: 4,
  PERFECT: 5
} as const;
```

### A-Factor Parameters
```typescript
const A_FACTOR = {
  MINIMUM: 1.1,
  MAXIMUM: 2.5,
  DEFAULT: 2.5,           // For new cards
  PRECISION: 0.01,        // Store as DECIMAL(3,2)
  CATEGORY_WIDTH: 0.1     // For OF Matrix discretization
} as const;
```

### Interval Constraints
```typescript
const INTERVALS = {
  MINIMUM_DAYS: 1,
  MAXIMUM_DAYS: 5475,     // 15 years
  GRACE_PERIOD: 0.2,      // ±20% of planned interval
  OVERDUE_THRESHOLD: 1.5, // 150% of planned interval
  EARLY_PENALTY: 0.8      // Reduced effectiveness for early reviews
} as const;
```

### Forgetting Index Targets
```typescript
const FORGETTING_INDEX = {
  TARGET: 0.1,            // 10% forgetting rate goal
  ACCEPTABLE_RANGE: {
    MINIMUM: 0.05,        // 5% (too easy)
    MAXIMUM: 0.2          // 20% (too hard)
  }
} as const;
```

## 🧮 Core Mathematical Formulas

### 1. Interval Calculation
```typescript
// First repetition
const calculateFirstInterval = (lapsesCount: number): number => {
  return getOptimalFactor(1, lapsesCount + 1);
};

// Subsequent repetitions  
const calculateNextInterval = (
  previousInterval: number,
  repetitionNumber: number,
  aFactor: number
): number => {
  const optimalFactor = getOptimalFactor(repetitionNumber, aFactor);
  return previousInterval * optimalFactor;
};
```

### 2. A-Factor Update Formula
```typescript
const updateAFactor = (
  currentAF: number,
  grade: number,
  interval: number
): number => {
  const expectedFI = calculateExpectedFI(interval, currentAF);
  const actualPerformance = gradeToPerformance(grade);
  const estimatedAF = calculateEstimatedAF(expectedFI, actualPerformance);
  
  const weightOld = getAdaptationWeight(interval);
  const weightNew = 1.0 - weightOld;
  
  const newAF = (currentAF * weightOld + estimatedAF * weightNew);
  
  return clamp(newAF, A_FACTOR.MINIMUM, A_FACTOR.MAXIMUM);
};
```

### 3. Memory Model Formulas
```typescript
// Retrievability calculation
const calculateRetrievability = (timeSinceReview: number, stability: number): number => {
  return Math.exp(-timeSinceReview / stability);
};

// Forgetting curve
const calculateForgettingProbability = (
  time: number,
  halfLife: number
): number => {
  return Math.exp(-(time * Math.log(2)) / halfLife);
};
```

### 4. Grade-to-Performance Mapping
```typescript
const GRADE_PERFORMANCE_MAP = {
  1: 0.0,   // Complete failure
  2: 0.3,   // Poor recall  
  3: 0.6,   // Moderate recall
  4: 0.8,   // Good recall
  5: 1.0    // Perfect recall
} as const;

const gradeToPerformance = (grade: number): number => {
  return GRADE_PERFORMANCE_MAP[grade as keyof typeof GRADE_PERFORMANCE_MAP] ?? 0.6;
};
```

## 📊 Default OF Matrix Values

### Research-Based Initial Matrix
```typescript
const DEFAULT_OF_MATRIX = {
  // [repetition_number][difficulty_category] = optimal_factor
  1: { 1: 1.30, 5: 1.30, 10: 1.30, 14: 1.30 }, // Easy start for all
  2: { 1: 1.70, 5: 1.80, 10: 1.85, 14: 2.00 }, // Moderate growth
  3: { 1: 1.80, 5: 1.90, 10: 2.00, 14: 2.20 }, // Steady increase
  4: { 1: 1.90, 5: 2.05, 10: 2.18, 14: 2.45 }, // Larger gaps
  5: { 1: 2.00, 5: 2.15, 10: 2.35, 14: 2.70 }  // Maximum growth
} as const;

const getDefaultOptimalFactor = (
  repetitionNumber: number,
  difficultyCategory: number
): number => {
  const baseFactors = [1.3, 1.85, 2.0, 2.18, 2.35];
  const difficultyAdjustment = 1.0 + (difficultyCategory * 0.05);
  
  if (repetitionNumber <= baseFactors.length) {
    return baseFactors[repetitionNumber - 1] * difficultyAdjustment;
  }
  
  // For very high repetition numbers
  return 2.5 * difficultyAdjustment;
};
```

## ⚙️ Algorithm Configuration Parameters

### Adaptation Weights
```typescript
const calculateAdaptationWeight = (intervalDays: number): number => {
  // Longer intervals get higher weight to historical data
  return Math.min(0.9, 0.3 + (intervalDays / 365) * 0.6);
};

const ADAPTATION_WEIGHTS = {
  NEW_INFORMATION: {
    SHORT_INTERVAL: 0.7,    // < 7 days: Trust new data more
    MEDIUM_INTERVAL: 0.3,   // 7-30 days: Balanced
    LONG_INTERVAL: 0.1      // > 30 days: Trust history more
  },
  HISTORICAL_DATA: {
    SHORT_INTERVAL: 0.3,
    MEDIUM_INTERVAL: 0.7,
    LONG_INTERVAL: 0.9
  }
} as const;
```

### Grade-Based Interval Adjustments
```typescript
const GRADE_MULTIPLIERS = {
  1: 0.2,   // Failed: Reset to very short interval
  2: 0.6,   // Hard: Significantly reduce interval  
  3: 0.8,   // Good: Slightly reduce interval
  4: 1.2,   // Easy: Increase interval
  5: 1.3    // Perfect: Significantly increase interval
} as const;

const adjustIntervalForGrade = (interval: number, grade: number): number => {
  const multiplier = GRADE_MULTIPLIERS[grade as keyof typeof GRADE_MULTIPLIERS] ?? 1.0;
  return interval * multiplier;
};
```

## 🧪 Testing Constants

### Test Data Ranges
```typescript
const TEST_CONSTANTS = {
  VALID_GRADES: [1, 2, 3, 4, 5],
  INVALID_GRADES: [0, 6, -1, 3.5, null, undefined],
  
  VALID_A_FACTORS: [1.1, 1.5, 2.0, 2.5],
  INVALID_A_FACTORS: [1.0, 2.6, -1, null],
  
  VALID_INTERVALS: [1, 5, 30, 365, 5475],
  INVALID_INTERVALS: [0, -1, 10000, null],
  
  SAMPLE_CARDS: {
    NEW: { aFactor: 2.5, repetitionCount: 0, intervalDays: 1 },
    EASY: { aFactor: 1.3, repetitionCount: 10, intervalDays: 45 },
    HARD: { aFactor: 2.8, repetitionCount: 5, intervalDays: 2 },
    AVERAGE: { aFactor: 2.0, repetitionCount: 3, intervalDays: 7 }
  }
} as const;
```

### Performance Benchmarks
```typescript
const PERFORMANCE_TARGETS = {
  RETENTION_RATE: {
    EXCELLENT: 0.95,   // 95%+ retention
    GOOD: 0.90,        // 90%+ retention  
    ACCEPTABLE: 0.85,  // 85%+ retention
    POOR: 0.80         // Below 80% needs tuning
  },
  
  INTERVAL_GROWTH: {
    OPTIMAL_MULTIPLIER: 2.1,     // Average growth per review
    MINIMUM_GROWTH: 1.5,         // Below this indicates stagnation
    MAXIMUM_GROWTH: 3.0          // Above this might be too aggressive
  },
  
  RESPONSE_TIMES: {
    GRADE_5: { max: 2000 },      // Perfect recall < 2s
    GRADE_4: { max: 3000 },      // Easy recall < 3s
    GRADE_3: { max: 5000 },      // Good recall < 5s
    GRADE_2: { max: 10000 },     // Hard recall < 10s
    GRADE_1: { timeout: 15000 }  // Give up after 15s
  }
} as const;
```

## 💾 Database Constraints

### Field Specifications
```typescript
const DATABASE_CONSTRAINTS = {
  A_FACTOR: {
    type: 'DECIMAL(3,2)',
    minimum: 1.10,
    maximum: 2.50,
    default: 2.50
  },
  
  INTERVAL_DAYS: {
    type: 'INTEGER',
    minimum: 1,
    maximum: 5475,
    default: 1
  },
  
  GRADE: {
    type: 'INTEGER',
    minimum: 1,
    maximum: 5,
    checkConstraint: 'grade BETWEEN 1 AND 5'
  },
  
  OPTIMAL_FACTOR: {
    type: 'DECIMAL(4,3)',
    minimum: 1.000,
    maximum: 3.000,
    precision: 0.001
  }
} as const;
```

### Index Requirements
```typescript
const REQUIRED_INDEXES = [
  'cards(user_id, next_review_date)',     // Critical for due cards
  'reviews(user_id, review_date DESC)',   // User activity timeline
  'reviews(card_id, review_date)',        // Card performance history
  'of_matrix(user_id, repetition_number, difficulty_category)', // OF lookups
  'user_statistics(user_id, date DESC)'   // Statistics queries
] as const;
```

## 🔧 Utility Functions

### Helper Calculations
```typescript
// Convert A-Factor to discrete category for OF Matrix
const afactorToCategory = (aFactor: number): number => {
  return Math.floor((aFactor - A_FACTOR.MINIMUM) / A_FACTOR.CATEGORY_WIDTH);
};

// Clamp value to range
const clamp = (value: number, min: number, max: number): number => {
  return Math.max(min, Math.min(max, value));
};

// Safe floating point comparison
const floatEquals = (a: number, b: number, epsilon: number = 0.001): boolean => {
  return Math.abs(a - b) < epsilon;
};

// Calculate days between dates
const daysBetween = (date1: Date, date2: Date): number => {
  const millisecondsPerDay = 24 * 60 * 60 * 1000;
  const timeDiff = Math.abs(date2.getTime() - date1.getTime());
  return Math.floor(timeDiff / millisecondsPerDay);
};
```

## 📈 Monitoring Thresholds

### Algorithm Health Indicators
```typescript
const HEALTH_THRESHOLDS = {
  // Warning signs that need attention
  WARNINGS: {
    LOW_RETENTION: 0.80,         // < 80% retention rate
    HIGH_FAILURE_RATE: 0.20,     // > 20% grade 1-2 reviews
    STAGNANT_INTERVALS: 0.05,    // > 5% cards not progressing
    OSCILLATING_CARDS: 0.10      // > 10% highly variable intervals
  },
  
  // Critical issues requiring immediate action
  CRITICAL: {
    VERY_LOW_RETENTION: 0.70,    // < 70% retention rate
    BROKEN_ALGORITHM: 0.50,      // < 50% retention indicates bugs
    MASS_FAILURES: 0.40          // > 40% failures suggests system issues
  }
} as const;
```

---

**🔗 Reference Links**:
- [Algorithm Specification](./algorithm-specification.md) - Mathematical framework
- [Implementation Examples](./implementation-examples.md) - Practical applications
- [Common Pitfalls](./common-pitfalls.md) - Implementation mistakes to avoid