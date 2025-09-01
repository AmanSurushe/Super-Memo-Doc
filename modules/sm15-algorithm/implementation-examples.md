# SM-15 Implementation Examples

Real-world examples and complete learning journeys that demonstrate how the SM-15 algorithm works in practice.

## Complete Learning Journey: "Buenos días" = "Good morning"

This example follows a single card through multiple review cycles, showing how the algorithm adapts.

### Day 1: Initial Card Creation
```typescript
const newCard = {
  front: "Buenos días",
  back: "Good morning",
  aFactor: 2.5,        // Default for new cards  
  repetitionCount: 0,   // Never reviewed
  intervalDays: 1,      // Show tomorrow
  lapsesCount: 0       // No failures yet
};
```

**Algorithm State**: Card created with neutral difficulty and scheduled for next day.

### Day 2: First Review - Grade 4 (Easy)
```typescript
// User sees card and quickly recalls "Good morning"
const reviewResult = calculateNextInterval(newCard, 4);

// Algorithm calculations:
// - A-Factor: 2.5 → 2.3 (card seems easier)
// - Interval: 1 → 3 days (next review in 3 days)  
// - Repetition Count: 0 → 1

const updatedCard = {
  ...newCard,
  aFactor: 2.3,
  repetitionCount: 1,
  intervalDays: 3,
  nextReviewDate: new Date('2024-01-05') // 3 days later
};
```

**Key Learning**: Quick recall indicates card is easier than default, so A-Factor decreases and interval grows moderately.

### Day 5: Second Review - Grade 3 (Good)  
```typescript
// User takes a moment to recall but gets it right
const secondReview = calculateNextInterval(updatedCard, 3);

// Algorithm calculations:
// - A-Factor: 2.3 → 2.35 (slightly harder than expected)
// - Interval: 3 → 6 days (still growing, but moderated)
// - Repetition Count: 1 → 2

const secondUpdatedCard = {
  ...updatedCard,
  aFactor: 2.35,
  repetitionCount: 2,
  intervalDays: 6,
  nextReviewDate: new Date('2024-01-11') // 6 days later
};
```

**Key Learning**: Slower recall suggests card is slightly harder than the previous review indicated.

### Day 11: Third Review - Grade 1 (Failed)
```typescript  
// User can't recall the answer at all
const failureReview = calculateNextInterval(secondUpdatedCard, 1);

// Algorithm calculations:
// - A-Factor: 2.35 → 2.7 (much harder now)
// - Interval: 6 → 1 day (back to daily review)
// - Lapses Count: 0 → 1 (failure recorded)
// - Repetition Count: 2 → 3

const failedCard = {
  ...secondUpdatedCard,
  aFactor: 2.7,
  repetitionCount: 3,
  intervalDays: 1,
  lapsesCount: 1,
  nextReviewDate: new Date('2024-01-12') // Tomorrow
};
```

**Key Learning**: Complete failure triggers algorithm reset - card is much harder than thought, needs daily drilling.

### Day 12: Fourth Review - Grade 5 (Perfect)
```typescript
// User has learned from mistake and recalls perfectly
const recoveryReview = calculateNextInterval(failedCard, 5);

// Algorithm calculations:  
// - A-Factor: 2.7 → 2.4 (easier again, but still cautious)
// - Interval: 1 → 2 days (gradual recovery)
// - Repetition Count: 3 → 4

const recoveredCard = {
  ...failedCard,
  aFactor: 2.4,
  repetitionCount: 4,
  intervalDays: 2,
  nextReviewDate: new Date('2024-01-14') // 2 days later
};
```

**Key Learning**: Perfect recall after failure shows learning occurred, but algorithm remains cautious with gradual interval increase.

## Algorithm Behavior Patterns

### Pattern 1: Consistent Performance (Easy Cards)
```typescript
// Card: "Two + Two = ?"
// Grade sequence: [5, 5, 4, 5, 4]
// Interval progression: 1 → 3 → 8 → 20 → 45 days
// A-Factor progression: 2.5 → 2.3 → 2.1 → 1.9 → 1.7

// Result: Card becomes "easy" with long intervals
```

### Pattern 2: Struggling Then Mastery  
```typescript
// Card: "What is the capital of Mongolia?"
// Grade sequence: [2, 1, 2, 3, 4, 5]
// Interval progression: 1 → 1 → 1 → 2 → 4 → 10 days
// A-Factor progression: 2.5 → 2.8 → 3.0 → 2.9 → 2.7 → 2.5

// Result: Difficult start, but eventually stabilizes
```

### Pattern 3: Inconsistent Performance
```typescript
// Card: "Translate: 'Serendipity'"
// Grade sequence: [4, 2, 5, 1, 3]
// Interval progression: 1 → 3 → 2 → 6 → 1 → 3 days  
// A-Factor progression: 2.5 → 2.3 → 2.6 → 2.2 → 2.8 → 2.7

// Result: A-Factor oscillates, intervals remain short due to inconsistency
```

## Real-World Grade Examples

### Grade 5 (Perfect) - Immediate Recall
```typescript
// Scenario: Card shows "Cat"
// User response: "Gato" (immediate, confident)
// Response time: < 1 second
// Algorithm effect: Large interval increase, A-Factor decrease

// Example progression:
// Current interval: 14 days → Next interval: 25 days (80% increase)
// A-Factor: 2.3 → 2.0 (significant difficulty decrease)
```

### Grade 4 (Easy) - Quick Recall  
```typescript
// Scenario: Card shows "Water"  
// User response: "Agua" (after 1-2 seconds of thought)
// Response time: 1-2 seconds
// Algorithm effect: Moderate interval increase, slight A-Factor decrease

// Example progression:
// Current interval: 7 days → Next interval: 12 days (50% increase)
// A-Factor: 2.0 → 1.9 (slight difficulty decrease)
```

### Grade 3 (Good) - Standard Recall
```typescript
// Scenario: Card shows "House"
// User response: "Casa... I think?" (hesitant but correct)
// Response time: 3-5 seconds  
// Algorithm effect: Small interval increase, minimal A-Factor change

// Example progression:
// Current interval: 5 days → Next interval: 6 days (20% increase)
// A-Factor: 2.0 → 2.05 (slight difficulty increase)
```

### Grade 2 (Hard) - Difficult Recall
```typescript  
// Scenario: Card shows "Library"
// User response: "Umm... biblioteca?" (long pause, uncertain)
// Response time: 10+ seconds
// Algorithm effect: Interval decrease, A-Factor increase

// Example progression:  
// Current interval: 10 days → Next interval: 6 days (40% decrease)
// A-Factor: 2.0 → 2.3 (moderate difficulty increase)
```

### Grade 1 (Failed) - No Recall
```typescript
// Scenario: Card shows "Dog" 
// User response: "I have no idea" (gives up)
// Response time: N/A (couldn't recall)
// Algorithm effect: Reset to daily review, significant A-Factor increase

// Example progression:
// Current interval: 20 days → Next interval: 1 day (reset)  
// A-Factor: 2.0 → 2.6 (significant difficulty increase)
// Lapses count: +1
```

## OF Matrix in Action

### Matrix Lookup Example
```typescript
// User reviews card with:
// - Repetition Count: 3 (third review)
// - A-Factor: 2.1 (moderately difficult)

// Step 1: Convert A-Factor to category
difficultyCategory = Math.floor((2.1 - 1.1) / 0.1) = 10

// Step 2: Query OF Matrix
// OF[repetitionNumber=3, difficultyCategory=10] = 2.15

// Step 3: Calculate new interval
// Previous interval was 5 days
newInterval = 5 × 2.15 = 10.75 ≈ 11 days
```

### Matrix Learning Example
```typescript
// Before user review:
// OF[3, 10] = 2.15 (predicted optimal factor)

// After user grades card as 2 (Hard):
// Algorithm realizes interval was too long
// Performance ratio = actualPerformance(0.3) / target(0.9) = 0.33
// New optimal factor = 2.15 × 0.33 = 0.71

// Updated matrix:
// OF[3, 10] = 0.71 (learned from experience)
// Future cards in this category will get shorter intervals
```

## Memory Model Examples

### Stable Memory (Easy Card)
```typescript
// Card: "Red" = "Rojo"
// Memory Stability: 12 days (well learned)

// Retrievability over time:
// Day 0: R = exp(0/12) = 1.00 (100% - just reviewed)  
// Day 3: R = exp(-3/12) = 0.78 (78% chance)
// Day 6: R = exp(-6/12) = 0.61 (61% chance) 
// Day 9: R = exp(-9/12) = 0.47 (47% chance)
// Day 12: R = exp(-12/12) = 0.37 (37% chance)

// Optimal review: Around day 10-11 (when R ≈ 0.4)
```

### Unstable Memory (Difficult Card)
```typescript  
// Card: "Serendipity" = "Serendipia"
// Memory Stability: 2 days (poorly learned)

// Retrievability over time:
// Day 0: R = exp(0/2) = 1.00 (100% - just reviewed)
// Day 1: R = exp(-1/2) = 0.61 (61% chance)
// Day 2: R = exp(-2/2) = 0.37 (37% chance) 
// Day 3: R = exp(-3/2) = 0.22 (22% chance)

// Optimal review: Around day 1-2 (when R ≈ 0.4-0.6)
```

## Common Learning Scenarios

### Scenario 1: Language Learning (Spanish Vocabulary)
```typescript
// Typical progression for language flashcards:
const spanishWords = [
  { word: "Hola", difficulty: "easy", finalInterval: "45 days" },
  { word: "Casa", difficulty: "easy", finalInterval: "30 days" }, 
  { word: "Biblioteca", difficulty: "medium", finalInterval: "12 days" },
  { word: "Serendipia", difficulty: "hard", finalInterval: "3 days" }
];

// Algorithm adapts intervals based on word complexity
// Common words: Long intervals (weeks/months)
// Complex words: Short intervals (days/week)  
```

### Scenario 2: Medical Studies (Anatomy Terms)
```typescript
// Medical flashcards tend to have:
// - High initial difficulty (A-Factor ≈ 2.7)
// - Gradual improvement over time
// - Long-term retention focus

// Example: "Function of the hippocampus"
// Week 1: Daily review (struggling)
// Week 2: Every 2-3 days (improving)
// Month 1: Weekly review (stabilizing)
// Month 3: Monthly review (mastered)
```

### Scenario 3: Programming Concepts
```typescript
// Programming flashcards show:
// - Variable difficulty based on complexity
// - Quick initial learning (high Grade 4-5)
// - Long stable intervals once understood

// Example: "What is a closure in JavaScript?"
// Day 1: Grade 3 → 2 days
// Day 3: Grade 4 → 5 days  
// Day 8: Grade 5 → 12 days
// Day 20: Grade 5 → 30 days (mastered)
```

## Algorithm Performance Metrics

### Success Indicators
```typescript
// Well-tuned SM-15 implementation shows:
const performanceMetrics = {
  retentionRate: "~90%",        // Users remember 9/10 cards
  averageAccuracy: "85-95%",    // Mostly grades 3-5
  intervalGrowth: "exponential", // Successful cards grow quickly
  timeToMastery: "2-8 weeks",   // Depending on difficulty
  reviewEfficiency: "high"      // Minimal time for maximum retention
};
```

---

**🔗 Next Steps**:
- Study [Common Pitfalls](./common-pitfalls.md) to avoid implementation mistakes
- Review [Algorithm Specification](./algorithm-specification.md) for mathematical details
- Check [Constants and Formulas](./constants-and-formulas.md) for reference values