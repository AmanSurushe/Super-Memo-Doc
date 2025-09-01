# SM-15 Algorithm Documentation

The SM-15 spaced repetition algorithm is the core of this learning application. This section provides comprehensive documentation for understanding and implementing the algorithm.

## 📋 Algorithm Overview

SM-15 is an advanced spaced repetition algorithm that optimizes learning by:
- **Adaptive Scheduling**: Cards appear at optimal intervals for maximum retention
- **Personalized Difficulty**: A-Factors adjust based on individual performance  
- **Dynamic Optimization**: OF Matrix learns from user patterns over time
- **Memory Science**: Based on forgetting curve and memory stability research

## 📚 Documentation Files

### Core Algorithm
- [**Algorithm Specification**](./algorithm-specification.md) - Mathematical framework and formulas
- [**Implementation Examples**](./implementation-examples.md) - Real-world examples and learning journeys
- [**Constants and Formulas**](./constants-and-formulas.md) - Algorithm parameters and equations

### Implementation Guidance
- [**Common Pitfalls**](./common-pitfalls.md) - Critical mistakes to avoid when implementing
- [**Pseudocode Reference**](./pseudocode-reference.md) - Step-by-step algorithm workflows

## 🔑 Key Concepts (Quick Reference)

### A-Factor System
- **Range**: 1.1 (easy) to 2.5 (hard)
- **Purpose**: Represents intrinsic item difficulty
- **Updates**: Weighted average of old and new difficulty estimates

### OF Matrix (Optimal Factors Matrix)
- **Structure**: 2D matrix `OF[repetition_number, difficulty_category]`  
- **Function**: Provides optimal interval multipliers
- **Learning**: Refines based on actual user performance

### Interval Calculation
```
I(1) = OF[1, L+1]           // First interval based on lapses
I(n) = I(n-1) × OF[n, AF]   // Subsequent intervals
```

### Memory Model
- **Stability (S)**: How long memory trace persists
- **Retrievability (R)**: Current recall probability
- **Forgetting Function**: `R = exp(-t/S)`

## 🎯 Grade Scale (5-Point System)

| Grade | Meaning | Effect | Example Response Time |
|-------|---------|--------|---------------------|
| 5 | Perfect | Significant interval increase | Immediate recall |
| 4 | Easy | Moderate interval increase | < 2 seconds |
| 3 | Good | Slight interval increase | 2-5 seconds |
| 2 | Hard | Interval decrease | 5+ seconds, hesitation |
| 1 | Failed | Reset to short interval | Could not recall |

## 🚀 Getting Started

1. **Start with [Algorithm Specification](./algorithm-specification.md)** for the mathematical foundation
2. **Read [Implementation Examples](./implementation-examples.md)** for practical understanding  
3. **Review [Common Pitfalls](./common-pitfalls.md)** to avoid critical mistakes
4. **Reference [Constants and Formulas](./constants-and-formulas.md)** during implementation

## 📊 Algorithm Performance

The SM-15 algorithm targets:
- **~90% retention rate** for reviewed cards
- **Optimal study efficiency** (minimal time, maximum retention)
- **Personalized learning curves** adapted to individual memory patterns
- **Long-term knowledge retention** with scientifically-based intervals

## 🔗 Related Documentation

- [Database Design](../database/README.md) - How algorithm data is stored
- [Backend Implementation](../backend/README.md) - SM-15 service implementation  
- [Frontend Implementation](../frontend/README.md) - User interface for reviews

---

**📘 Ready to dive deeper?** Start with the [Algorithm Specification](./algorithm-specification.md) to understand the mathematical framework behind SM-15.