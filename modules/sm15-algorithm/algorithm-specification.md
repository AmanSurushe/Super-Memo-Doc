# SM-15 Algorithm Specification

Complete mathematical framework and technical specifications for the SM-15 spaced repetition algorithm.

## Core Mathematical Framework

### Interval Calculation
**Primary Formula**: `I(n) = I(n-1) × OF[n, AF]`

**Components**:
- **I(n)**: nth inter-repetition interval (in days)
- **OF**: Matrix of Optimal Factors  
- **AF**: Absolute Difficulty Factor (A-Factor)
- **n**: Repetition number

### A-Factor System
**Purpose**: Dynamic difficulty tracking that replaces E-Factors from earlier algorithms

**Characteristics**:
- **Range**: 1.1 to 2.5
- **Initial Value**: 2.5 for new cards
- **Updates**: After each review using weighted averages
- **Meaning**: Higher = harder card = shorter intervals

### OF Matrix Structure
**Design**: 2D lookup table for optimal interval multipliers

**Dimensions**:
- **Rows**: Repetition numbers (1, 2, 3, ...)
- **Columns**: A-Factor ranges (difficulty categories)  
- **Values**: Optimal interval multipliers

### Memory Model
**Two-Component System**:
1. **Memory Stability (S)**: How long memory trace persists
2. **Memory Retrievability (R)**: Current recall probability

**Forgetting Function**: `R = exp(-t/S)`
- **R**: Retrievability (0-1)
- **t**: Time since last review  
- **S**: Memory stability

## Detailed Algorithm Specifications

### 1. Interval Calculation Formulas

#### First Repetition
```
I(1) = OF[1, L+1]
```
Where **L** = Number of memory lapses for the item

#### Subsequent Repetitions
```
I(n) = I(n-1) × OF[n, AF]
```

#### Real-World Example
```typescript
// Spanish card "Hola" = "Hello"
// Current state: repetitionCount = 2, aFactor = 2.0, lastInterval = 5 days

// First repetition (n=1):
I(1) = OF[1, L+1]  // L=0 (no lapses), so OF[1,1] = 1.3 days

// Second repetition (n=2):  
I(2) = I(1) × OF[2, AF]  // I(1)=1.3, AF=2.0, so OF[2,2.0] = 1.85
I(2) = 1.3 × 1.85 = 2.4 days

// Third repetition (n=3):
I(3) = I(2) × OF[3, AF]  // I(2)=2.4, AF=2.0, so OF[3,2.0] = 2.0
I(3) = 2.4 × 2.0 = 4.8 days ≈ 5 days
```

### 2. A-Factor Update Process

#### Update Formula
```
New_AF = (Old_AF × weight_old + Estimated_AF × weight_new) / (weight_old + weight_new)
```

#### Weight Calculation
- **weight_old**: Based on interval length (longer intervals = higher weight)
- **weight_new**: Typically `1.0 - weight_old`
- **Purpose**: Balance historical data with new performance evidence

#### Real-World Example
```typescript
// User struggles with difficult card
// Initial A-Factor = 2.5 (default for new cards)

// After grade 2 (hard recall):
Old_AF = 2.5, weight_old = 0.7, Estimated_AF = 2.8, weight_new = 0.3
New_AF = (2.5 × 0.7 + 2.8 × 0.3) / (0.7 + 0.3) = 2.59

// Card becomes harder → shorter intervals
// Prevents seeing difficult cards too infrequently

// After grade 5 (perfect recall):
Old_AF = 2.5, Estimated_AF = 2.2 (easier)  
New_AF = (2.5 × 0.7 + 2.2 × 0.3) / 1.0 = 2.41

// Card becomes easier → longer intervals allowed
```

### 3. OF Matrix (Optimal Factors Matrix)

#### Matrix Structure
```typescript
// Simplified 5×5 OF Matrix Example:
//                A-Factor Categories
//   Rep#    1.3   1.6   2.0   2.3   2.6  (Difficulty)
//    1     1.30  1.30  1.30  1.30  1.30  (Easy start)
//    2     1.70  1.80  1.85  1.90  2.00  (Moderate growth)
//    3     1.80  1.90  2.00  2.10  2.20  (Steady increase)  
//    4     1.90  2.05  2.18  2.30  2.45  (Larger gaps)
//    5     2.00  2.15  2.35  2.50  2.70  (Maximum growth)
```

#### Usage Example
```typescript
// Card with repetition #3 and A-Factor 2.0:
// Next interval = current_interval × OF[3, 2.0]
// Next interval = current_interval × 2.00

// If current interval is 7 days:
// Next interval = 7 × 2.00 = 14 days
```

#### Matrix Updates
- **Real-time refinement** after each repetition
- **Regression analysis** and power approximation
- **Performance feedback** accounts for timing deviations and grades

### 4. Two-Component Memory Model

#### Memory Stability (S)
- **Definition**: How long memory trace naturally persists
- **Units**: Days
- **Updates**: Increases with successful reviews, decreases with failures

#### Memory Retrievability (R)  
- **Definition**: Current probability of successful recall
- **Range**: 0 to 1 (0% to 100%)
- **Calculation**: `R = exp(-t/S)`

#### Intuitive Timeline Example
```typescript
// User learns "Gato" = "Cat" in Spanish

// Initial state:
Memory_Stability = 1.5 days    // Natural memory persistence
Time_since_review = 0 days     // Just learned

// Memory decay over time:
// After 1 day: R = exp(-1/1.5) = 0.51 (51% recall chance)
// After 2 days: R = exp(-2/1.5) = 0.26 (26% recall chance)  
// After 3 days: R = exp(-3/1.5) = 0.14 (14% recall chance)

// If reviewed at day 2 with success, S increases to ~2.8 days
// More stable memory for future:
// After 3 days: R = exp(-3/2.8) = 0.34 (34% recall chance)
```

### 5. Forgetting Curve Integration

#### Mathematical Model
```
P(recall) = exp(-(t × ln(2)) / half_life)
```

#### Implementation Steps
1. **Calculate expected forgetting index** for current interval
2. **Compare with actual performance** (grade given by user)
3. **Adjust memory parameters** based on performance difference
4. **Update OF matrix values** to improve future predictions

### 6. Grade-to-Performance Mapping

#### Standard 5-Point Scale
| Grade | Performance | Interval Effect | A-Factor Change |
|-------|-------------|-----------------|-----------------|
| 5 | Perfect (1.0) | +80% increase | Significant decrease |
| 4 | Good (0.8) | +50% increase | Moderate decrease |  
| 3 | Adequate (0.6) | +20% increase | Slight adjustment |
| 2 | Difficult (0.3) | -40% decrease | Moderate increase |
| 1 | Failed (0.0) | Reset to 1 day | Significant increase |

#### FI-G Graph (Forgetting Index vs Grade)
- **Purpose**: Maps grades to expected forgetting probabilities
- **Calibration**: Used to adjust interval calculations
- **Personalization**: Adapts based on individual user performance history

### 7. Algorithm Constants

#### Key Parameters
- **Requested Forgetting Index**: ~10% (0.1)
- **Minimum Interval**: 1 day
- **Maximum Interval**: ~15 years (5,475 days)
- **A-Factor Range**: 1.1 to 2.5
- **Grade Scale**: 1-5 (integer values only)

#### Timing Constants
- **Review Grace Period**: ±20% of interval
- **Overdue Threshold**: 150% of planned interval
- **Early Review Penalty**: Reduced interval effectiveness

## Implementation Considerations

### Precision Requirements
- **A-Factor**: Store as DECIMAL(3,2) for 0.01 precision
- **Intervals**: Integer days (minimum 1, maximum 5475)
- **OF Matrix Values**: DECIMAL(4,3) for 0.001 precision

### Edge Case Handling
- **Grade Validation**: Must be integer 1-5
- **A-Factor Clamping**: Always within 1.1-2.5 range
- **Interval Bounds**: Never below 1 day or above maximum
- **NULL Handling**: Default values for missing OF Matrix entries

### Performance Optimization
- **Matrix Caching**: Store frequently accessed OF values
- **Batch Updates**: Process multiple reviews efficiently
- **Index Strategy**: Optimize for due card queries

---

**🔗 Next Steps**: 
- Review [Implementation Examples](./implementation-examples.md) for practical applications
- Check [Common Pitfalls](./common-pitfalls.md) to avoid implementation mistakes
- See [Constants and Formulas](./constants-and-formulas.md) for reference values