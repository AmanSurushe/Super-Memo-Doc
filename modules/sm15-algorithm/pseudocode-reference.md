# SM-15 Algorithm Pseudocode Reference

Complete pseudocode implementation guide for the SM-15 spaced repetition algorithm, based on SuperMemo 15 specifications and research.

## Table of Contents

1. [Main Algorithm Flow](#main-algorithm-flow)
2. [Core Data Structures](#core-data-structures)
3. [Constants and Configuration](#constants-and-configuration)
4. [Interval Calculation](#interval-calculation)
5. [A-Factor Management](#a-factor-management)
6. [OF Matrix Operations](#of-matrix-operations)
7. [Memory Model Integration](#memory-model-integration)
8. [Grade Processing](#grade-processing)
9. [Helper Functions](#helper-functions)
10. [Error Handling](#error-handling)

## Main Algorithm Flow

```pseudocode
ALGORITHM SM15_ProcessReview
INPUT: 
    card: Card object
    grade: integer (1-5)
    reviewDate: Date
    responseTime: integer (milliseconds)
OUTPUT: 
    updatedCard: Card object with new scheduling data

BEGIN
    // Validate inputs
    IF NOT IsValidGrade(grade) THEN
        THROW InvalidGradeException
    END IF
    
    // Calculate time since last review
    timeSinceReview = CalculateDaysBetween(card.lastReviewDate, reviewDate)
    
    // Update review statistics
    card.totalReviews = card.totalReviews + 1
    card.lastReviewDate = reviewDate
    card.lastGrade = grade
    card.responseTime = responseTime
    
    // Handle failure case
    IF grade == 1 THEN
        card = HandleFailure(card)
    ELSE
        card = HandleSuccess(card, grade, timeSinceReview)
    END IF
    
    // Update repetition count
    card.repetitionCount = card.repetitionCount + 1
    
    // Calculate next review date
    card.nextReviewDate = CalculateNextReviewDate(reviewDate, card.intervalDays)
    
    // Update user's OF Matrix based on performance
    UpdateOFMatrix(card, grade, timeSinceReview)
    
    // Log review for analytics
    LogReview(card, grade, reviewDate, responseTime)
    
    RETURN card
END

ALGORITHM SM15_InitializeNewCard
INPUT: 
    cardContent: String
    userId: integer
OUTPUT: 
    newCard: Card object

BEGIN
    newCard = NEW Card
    newCard.userId = userId
    newCard.content = cardContent
    newCard.aFactor = DEFAULT_A_FACTOR  // 2.5
    newCard.repetitionCount = 0
    newCard.intervalDays = 1
    newCard.lapsesCount = 0
    newCard.totalReviews = 0
    newCard.creationDate = CurrentDate()
    newCard.nextReviewDate = CurrentDate() + 1  // Tomorrow
    newCard.lastReviewDate = NULL
    newCard.lastGrade = NULL
    newCard.averageGrade = NULL
    newCard.memoryStability = INITIAL_STABILITY  // 1.5 days
    
    RETURN newCard
END
```

## Core Data Structures

```pseudocode
STRUCTURE Card
    // Identification
    id: integer
    userId: integer
    content: string
    
    // Scheduling Data
    aFactor: decimal(3,2)           // 1.10 to 2.50
    repetitionCount: integer        // 0, 1, 2, 3, ...
    intervalDays: integer           // 1 to 5475
    lapsesCount: integer            // Number of failures
    nextReviewDate: Date
    lastReviewDate: Date
    
    // Performance Tracking
    totalReviews: integer
    lastGrade: integer              // 1-5
    averageGrade: decimal(2,1)      // Running average
    responseTime: integer           // Last response in milliseconds
    
    // Memory Model
    memoryStability: decimal(5,2)   // Days
    memoryRetrievability: decimal(3,2) // 0.00 to 1.00
    
    // Metadata
    creationDate: Date
    modificationDate: Date
END STRUCTURE

STRUCTURE OFMatrixEntry
    userId: integer
    repetitionNumber: integer       // 1, 2, 3, ...
    difficultyCategory: integer     // 1-25 (based on A-Factor)
    optimalFactor: decimal(4,3)     // 1.000 to 3.000
    sampleSize: integer             // Number of observations
    lastUpdated: Date
END STRUCTURE

STRUCTURE ReviewLog
    cardId: integer
    userId: integer
    reviewDate: Date
    grade: integer
    responseTime: integer
    intervalBefore: integer
    intervalAfter: integer
    aFactorBefore: decimal(3,2)
    aFactorAfter: decimal(3,2)
END STRUCTURE
```

## Constants and Configuration

```pseudocode
// Grade Scale
CONSTANT MIN_GRADE = 1
CONSTANT MAX_GRADE = 5
CONSTANT FAILED_GRADE = 1
CONSTANT HARD_GRADE = 2
CONSTANT GOOD_GRADE = 3
CONSTANT EASY_GRADE = 4
CONSTANT PERFECT_GRADE = 5

// A-Factor Parameters
CONSTANT MIN_A_FACTOR = 1.10
CONSTANT MAX_A_FACTOR = 2.50
CONSTANT DEFAULT_A_FACTOR = 2.50
CONSTANT A_FACTOR_CATEGORY_WIDTH = 0.10

// Interval Constraints
CONSTANT MIN_INTERVAL_DAYS = 1
CONSTANT MAX_INTERVAL_DAYS = 5475  // 15 years
CONSTANT FAILURE_RESET_INTERVAL = 1
CONSTANT GRACE_PERIOD_RATIO = 0.20  // ±20%

// Memory Model
CONSTANT INITIAL_STABILITY = 1.5    // Days for new cards
CONSTANT TARGET_FORGETTING_INDEX = 0.10  // 10%
CONSTANT MIN_RETRIEVABILITY = 0.05
CONSTANT MAX_RETRIEVABILITY = 1.00

// OF Matrix
CONSTANT MAX_REPETITION_NUMBER = 20
CONSTANT MAX_DIFFICULTY_CATEGORIES = 25
CONSTANT DEFAULT_OPTIMAL_FACTOR = 1.30

// Grade Performance Mapping
CONSTANT GRADE_PERFORMANCE = {
    1: 0.0,   // Complete failure
    2: 0.3,   // Poor recall
    3: 0.6,   // Moderate recall
    4: 0.8,   // Good recall
    5: 1.0    // Perfect recall
}
```

## Interval Calculation

```pseudocode
FUNCTION CalculateNextInterval
INPUT: 
    card: Card object
    grade: integer
OUTPUT: 
    newIntervalDays: integer

BEGIN
    IF card.repetitionCount == 0 THEN
        // First repetition - based on lapses
        newInterval = GetOptimalFactor(1, card.lapsesCount + 1)
    ELSE
        // Subsequent repetitions - use OF Matrix
        difficultyCategory = AFToDifficultyCategory(card.aFactor)
        optimalFactor = GetOptimalFactor(card.repetitionCount + 1, difficultyCategory)
        newInterval = card.intervalDays * optimalFactor
    END IF
    
    // Apply grade-based adjustment
    newInterval = ApplyGradeMultiplier(newInterval, grade)
    
    // Ensure within bounds
    newInterval = ClampInterval(newInterval)
    
    RETURN ROUND(newInterval)
END

FUNCTION ApplyGradeMultiplier
INPUT: 
    interval: decimal
    grade: integer
OUTPUT: 
    adjustedInterval: decimal

BEGIN
    SWITCH grade
        CASE 1: RETURN 1                    // Reset to daily
        CASE 2: RETURN interval * 0.60      // Significant reduction
        CASE 3: RETURN interval * 0.85      // Slight reduction
        CASE 4: RETURN interval * 1.15      // Slight increase
        CASE 5: RETURN interval * 1.30      // Significant increase
        DEFAULT: RETURN interval
    END SWITCH
END

FUNCTION ClampInterval
INPUT: 
    interval: decimal
OUTPUT: 
    clampedInterval: integer

BEGIN
    IF interval < MIN_INTERVAL_DAYS THEN
        RETURN MIN_INTERVAL_DAYS
    ELSE IF interval > MAX_INTERVAL_DAYS THEN
        RETURN MAX_INTERVAL_DAYS
    ELSE
        RETURN ROUND(interval)
    END IF
END
```

## A-Factor Management

```pseudocode
FUNCTION UpdateAFactor
INPUT: 
    card: Card object
    grade: integer
    timeSinceReview: integer
OUTPUT: 
    newAFactor: decimal

BEGIN
    // Calculate expected performance based on current A-Factor
    expectedFI = CalculateExpectedForgettingIndex(card.intervalDays, card.aFactor)
    
    // Map grade to actual performance
    actualPerformance = GRADE_PERFORMANCE[grade]
    
    // Estimate new A-Factor based on performance difference
    performanceRatio = actualPerformance / (1.0 - expectedFI)
    estimatedAF = card.aFactor / performanceRatio
    
    // Calculate adaptation weights based on interval length
    weightOld = CalculateAdaptationWeight(card.intervalDays)
    weightNew = 1.0 - weightOld
    
    // Weighted update
    newAFactor = (card.aFactor * weightOld + estimatedAF * weightNew)
    
    // Ensure within bounds
    newAFactor = ClampAFactor(newAFactor)
    
    RETURN newAFactor
END

FUNCTION CalculateAdaptationWeight
INPUT: 
    intervalDays: integer
OUTPUT: 
    weight: decimal

BEGIN
    // Longer intervals get higher weight to historical data
    IF intervalDays <= 7 THEN
        RETURN 0.30  // Trust new information more
    ELSE IF intervalDays <= 30 THEN
        RETURN 0.60  // Balanced
    ELSE
        RETURN 0.85  // Trust historical data more
    END IF
END

FUNCTION ClampAFactor
INPUT: 
    aFactor: decimal
OUTPUT: 
    clampedAFactor: decimal

BEGIN
    IF aFactor < MIN_A_FACTOR THEN
        RETURN MIN_A_FACTOR
    ELSE IF aFactor > MAX_A_FACTOR THEN
        RETURN MAX_A_FACTOR
    ELSE
        RETURN ROUND(aFactor, 2)  // Round to 2 decimal places
    END IF
END

FUNCTION AFToDifficultyCategory
INPUT: 
    aFactor: decimal
OUTPUT: 
    category: integer

BEGIN
    category = FLOOR((aFactor - MIN_A_FACTOR) / A_FACTOR_CATEGORY_WIDTH)
    RETURN MIN(category, MAX_DIFFICULTY_CATEGORIES - 1)
END
```

## OF Matrix Operations

```pseudocode
FUNCTION GetOptimalFactor
INPUT: 
    repetitionNumber: integer
    difficultyCategory: integer
OUTPUT: 
    optimalFactor: decimal

BEGIN
    // Try to get from user's personalized matrix
    factor = QueryOFMatrix(currentUserId, repetitionNumber, difficultyCategory)
    
    IF factor IS NOT NULL THEN
        RETURN factor
    ELSE
        // Fall back to default matrix
        RETURN GetDefaultOptimalFactor(repetitionNumber, difficultyCategory)
    END IF
END

FUNCTION GetDefaultOptimalFactor
INPUT: 
    repetitionNumber: integer
    difficultyCategory: integer
OUTPUT: 
    optimalFactor: decimal

BEGIN
    // Base factors for different repetition numbers
    baseFactor = 1.30  // Conservative default
    
    SWITCH repetitionNumber
        CASE 1: baseFactor = 1.30
        CASE 2: baseFactor = 1.85
        CASE 3: baseFactor = 2.00
        CASE 4: baseFactor = 2.18
        CASE 5: baseFactor = 2.35
        DEFAULT: baseFactor = 2.50  // For high repetition numbers
    END SWITCH
    
    // Adjust based on difficulty
    difficultyMultiplier = 1.0 + (difficultyCategory * 0.05)
    
    RETURN baseFactor * difficultyMultiplier
END

FUNCTION UpdateOFMatrix
INPUT: 
    card: Card object
    grade: integer
    timeSinceReview: integer
OUTPUT: 
    none (updates matrix in database)

BEGIN
    // Calculate actual vs expected performance
    expectedFI = CalculateExpectedForgettingIndex(card.intervalDays, card.aFactor)
    actualPerformance = GRADE_PERFORMANCE[grade]
    
    // Calculate what the optimal factor should have been
    IF actualPerformance > 0 THEN
        performanceRatio = actualPerformance / (1.0 - expectedFI)
        optimalFactor = (card.intervalDays / card.previousInterval) / performanceRatio
    ELSE
        optimalFactor = 0.5  // For complete failures
    END IF
    
    // Update the matrix entry
    difficultyCategory = AFToDifficultyCategory(card.aFactor)
    UpdateMatrixEntry(card.userId, card.repetitionCount, difficultyCategory, optimalFactor)
END

FUNCTION UpdateMatrixEntry
INPUT: 
    userId: integer
    repetitionNumber: integer
    difficultyCategory: integer
    newOptimalFactor: decimal
OUTPUT: 
    none

BEGIN
    entry = GetMatrixEntry(userId, repetitionNumber, difficultyCategory)
    
    IF entry IS NULL THEN
        // Create new entry
        entry = NEW OFMatrixEntry
        entry.userId = userId
        entry.repetitionNumber = repetitionNumber
        entry.difficultyCategory = difficultyCategory
        entry.optimalFactor = newOptimalFactor
        entry.sampleSize = 1
    ELSE
        // Update existing entry with exponential moving average
        alpha = 0.1  // Learning rate
        entry.optimalFactor = entry.optimalFactor * (1 - alpha) + newOptimalFactor * alpha
        entry.sampleSize = entry.sampleSize + 1
    END IF
    
    entry.lastUpdated = CurrentDate()
    SaveMatrixEntry(entry)
END
```

## Memory Model Integration

```pseudocode
FUNCTION UpdateMemoryModel
INPUT: 
    card: Card object
    grade: integer
    timeSinceReview: integer
OUTPUT: 
    updatedCard: Card object

BEGIN
    // Calculate current retrievability
    currentRetrievability = CalculateRetrievability(timeSinceReview, card.memoryStability)
    
    // Update stability based on review outcome
    IF grade >= 3 THEN
        // Successful recall - increase stability
        stabilityIncrease = card.memoryStability * 0.2 * (grade / 5.0)
        card.memoryStability = card.memoryStability + stabilityIncrease
    ELSE
        // Failed recall - decrease stability
        stabilityDecrease = card.memoryStability * 0.3
        card.memoryStability = MAX(card.memoryStability - stabilityDecrease, 1.0)
    END IF
    
    // Update retrievability for next calculation
    card.memoryRetrievability = 1.0  // Reset to 100% after review
    
    RETURN card
END

FUNCTION CalculateRetrievability
INPUT: 
    timeSinceReview: integer
    stability: decimal
OUTPUT: 
    retrievability: decimal

BEGIN
    IF stability <= 0 THEN
        RETURN 0.0
    END IF
    
    // Exponential decay function: R = exp(-t/S)
    retrievability = EXP(-timeSinceReview / stability)
    
    RETURN CLAMP(retrievability, 0.0, 1.0)
END

FUNCTION CalculateExpectedForgettingIndex
INPUT: 
    intervalDays: integer
    aFactor: decimal
OUTPUT: 
    forgettingIndex: decimal

BEGIN
    // Use A-Factor to estimate memory stability
    estimatedStability = intervalDays / LOG(1.0 / TARGET_FORGETTING_INDEX)
    
    // Calculate expected forgetting at end of interval
    forgettingIndex = 1.0 - EXP(-intervalDays / estimatedStability)
    
    RETURN CLAMP(forgettingIndex, 0.01, 0.95)
END
```

## Grade Processing

```pseudocode
FUNCTION HandleFailure
INPUT: 
    card: Card object
OUTPUT: 
    updatedCard: Card object

BEGIN
    // Reset to daily review
    card.intervalDays = FAILURE_RESET_INTERVAL
    
    // Increase difficulty (A-Factor)
    aFactorIncrease = 0.2 + (0.1 * card.lapsesCount)
    card.aFactor = ClampAFactor(card.aFactor + aFactorIncrease)
    
    // Increment lapse count
    card.lapsesCount = card.lapsesCount + 1
    
    // Update memory model
    card = UpdateMemoryModel(card, 1, 0)
    
    RETURN card
END

FUNCTION HandleSuccess
INPUT: 
    card: Card object
    grade: integer
    timeSinceReview: integer
OUTPUT: 
    updatedCard: Card object

BEGIN
    // Update A-Factor based on performance
    card.aFactor = UpdateAFactor(card, grade, timeSinceReview)
    
    // Calculate new interval
    card.intervalDays = CalculateNextInterval(card, grade)
    
    // Update memory model
    card = UpdateMemoryModel(card, grade, timeSinceReview)
    
    // Update running averages
    UpdateCardStatistics(card, grade)
    
    RETURN card
END

FUNCTION UpdateCardStatistics
INPUT: 
    card: Card object
    grade: integer
OUTPUT: 
    none (updates card statistics)

BEGIN
    // Update average grade with exponential moving average
    IF card.averageGrade IS NULL THEN
        card.averageGrade = grade
    ELSE
        alpha = 0.1
        card.averageGrade = card.averageGrade * (1 - alpha) + grade * alpha
    END IF
    
    // Update other statistics as needed
    card.modificationDate = CurrentDate()
END
```

## Helper Functions

```pseudocode
FUNCTION IsValidGrade
INPUT: 
    grade: integer
OUTPUT: 
    isValid: boolean

BEGIN
    RETURN grade >= MIN_GRADE AND grade <= MAX_GRADE
END

FUNCTION CalculateDaysBetween
INPUT: 
    date1: Date
    date2: Date
OUTPUT: 
    days: integer

BEGIN
    millisecondsDiff = ABS(date2.getTime() - date1.getTime())
    daysDiff = FLOOR(millisecondsDiff / (24 * 60 * 60 * 1000))
    RETURN daysDiff
END

FUNCTION CalculateNextReviewDate
INPUT: 
    currentDate: Date
    intervalDays: integer
OUTPUT: 
    nextReviewDate: Date

BEGIN
    RETURN currentDate + intervalDays
END

FUNCTION LogReview
INPUT: 
    card: Card object
    grade: integer
    reviewDate: Date
    responseTime: integer
OUTPUT: 
    none

BEGIN
    reviewLog = NEW ReviewLog
    reviewLog.cardId = card.id
    reviewLog.userId = card.userId
    reviewLog.reviewDate = reviewDate
    reviewLog.grade = grade
    reviewLog.responseTime = responseTime
    reviewLog.intervalBefore = card.previousInterval
    reviewLog.intervalAfter = card.intervalDays
    reviewLog.aFactorBefore = card.previousAFactor
    reviewLog.aFactorAfter = card.aFactor
    
    SaveReviewLog(reviewLog)
END
```

## Error Handling

```pseudocode
FUNCTION ValidateCard
INPUT: 
    card: Card object
OUTPUT: 
    isValid: boolean, errors: Array<String>

BEGIN
    errors = []
    
    IF card.aFactor < MIN_A_FACTOR OR card.aFactor > MAX_A_FACTOR THEN
        errors.ADD("Invalid A-Factor: " + card.aFactor)
    END IF
    
    IF card.intervalDays < MIN_INTERVAL_DAYS OR card.intervalDays > MAX_INTERVAL_DAYS THEN
        errors.ADD("Invalid interval: " + card.intervalDays)
    END IF
    
    IF card.repetitionCount < 0 THEN
        errors.ADD("Invalid repetition count: " + card.repetitionCount)
    END IF
    
    IF card.lapsesCount < 0 THEN
        errors.ADD("Invalid lapses count: " + card.lapsesCount)
    END IF
    
    RETURN (errors.LENGTH == 0), errors
END

EXCEPTION InvalidGradeException
    message: "Grade must be between 1 and 5"
END EXCEPTION

EXCEPTION InvalidCardException
    message: "Card data is invalid"
    errors: Array<String>
END EXCEPTION

EXCEPTION OFMatrixException
    message: "Error accessing OF Matrix"
END EXCEPTION
```

## Integration Points

```pseudocode
// Database Interface Functions (to be implemented)
FUNCTION QueryOFMatrix(userId, repetitionNumber, difficultyCategory) -> decimal
FUNCTION SaveMatrixEntry(entry: OFMatrixEntry) -> void  
FUNCTION GetMatrixEntry(userId, repetitionNumber, difficultyCategory) -> OFMatrixEntry
FUNCTION SaveCard(card: Card) -> void
FUNCTION SaveReviewLog(log: ReviewLog) -> void

// Configuration Interface
FUNCTION GetUserSettings(userId) -> UserSettings
FUNCTION GetSystemConfiguration() -> SystemConfig

// Analytics Interface  
FUNCTION RecordMetric(metricName, value, timestamp) -> void
FUNCTION UpdateUserStatistics(userId, card: Card, grade) -> void
```

---

## Usage Example

```pseudocode
// Example: Process a review session
BEGIN
    card = LoadCard(cardId)
    grade = GetUserGrade()  // 1-5 from user interface
    reviewDate = CurrentDate()
    responseTime = GetResponseTime()  // milliseconds
    
    TRY
        updatedCard = SM15_ProcessReview(card, grade, reviewDate, responseTime)
        SaveCard(updatedCard)
        
        PRINT "Next review: " + updatedCard.nextReviewDate
        PRINT "New interval: " + updatedCard.intervalDays + " days"
        
    CATCH InvalidGradeException
        PRINT "Please enter a grade between 1 and 5"
    CATCH InvalidCardException AS e
        PRINT "Card validation failed: " + e.errors
    END TRY
END
```

---

**🔗 Related Documentation**:
- [Algorithm Specification](./algorithm-specification.md) - Mathematical framework
- [Implementation Examples](./implementation-examples.md) - Real-world usage scenarios  
- [Constants and Formulas](./constants-and-formulas.md) - Reference values and equations
- [Common Pitfalls](./common-pitfalls.md) - Implementation mistakes to avoid

---

*This pseudocode reference provides a complete implementation guide for the SM-15 algorithm. Use it as a blueprint for actual code implementation in your preferred programming language.*