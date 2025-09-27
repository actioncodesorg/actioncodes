# AIP-6: Strategy Pattern Refactoring

- **Author**: Action Codes Core Team
- **Status**: Draft
- **Created**: 2025-09-25

## Overview

This AIP proposes refactoring the current mode-based system (Mode W, Mode A) into a strategy pattern architecture for better code organization, unified validation, and improved extensibility while maintaining all existing functionality.

## Motivation

The current mode-based system has limitations:

- **Scattered Logic**: Code generation and validation logic is spread across multiple classes
- **Inconsistent Validation**: Different validation approaches for different modes
- **Hard to Extend**: Adding new authentication methods requires changes to core protocol
- **Complex API**: Multiple methods for similar functionality

A strategy pattern refactoring provides:

- **Unified Interface**: Single API for all authentication methods
- **Better Organization**: Clear separation of concerns
- **Easier Extension**: New strategies can be added without changing core protocol
- **Consistent Validation**: Unified validation context and process
- **Maintainability**: Cleaner, more maintainable code structure

## Specification

### Core Strategy Interface

```typescript
interface CodeGenerationStrategy {
    generateCode(...args: any[]): Promise<any>
    validateCode(...args: any[]): Promise<any>
    getStrategyType(): string
    getCapabilities(): any
}
```

### Supporting Interfaces

```typescript
interface ValidationContext {
    // Implementation-specific fields
}

interface CodeResult {
    // Implementation-specific fields
}

interface StrategyCapabilities {
    // Implementation-specific fields
}
```

## Strategy Implementations

### 1. Wallet Strategy (Mode W)

```typescript
class WalletStrategy implements CodeGenerationStrategy {
    constructor(...args: any[]) {}
    
    async generateCode(...args: any[]): Promise<any> {
        // Implementation-specific logic
    }
    
    async validateCode(...args: any[]): Promise<any> {
        // Implementation-specific logic
    }
    
    getStrategyType(): string {
        return 'wallet';
    }
    
    getCapabilities(): any {
        // Implementation-specific capabilities
    }
}
```

### 2. Delegated Strategy (Mode A)

```typescript
class DelegatedStrategy implements CodeGenerationStrategy {
    constructor(...args: any[]) {}
    
    async generateCode(...args: any[]): Promise<any> {
        // Implementation-specific logic
    }
    
    async validateCode(...args: any[]): Promise<any> {
        // Implementation-specific logic
    }
    
    getStrategyType(): string {
        return 'delegated';
    }
    
    getCapabilities(): any {
        // Implementation-specific capabilities
    }
}
```

## Unified Protocol API

```typescript
class ActionCodesProtocol {
    constructor(private strategy: CodeGenerationStrategy) {}
    
    async generateCode(...args: any[]): Promise<any> {
        return this.strategy.generateCode(...args);
    }
    
    async validateCode(...args: any[]): Promise<any> {
        return this.strategy.validateCode(...args);
    }
    
    setStrategy(strategy: CodeGenerationStrategy): void {
        this.strategy = strategy;
    }
    
    getCurrentStrategy(): CodeGenerationStrategy {
        return this.strategy;
    }
}
```

## Benefits

- **Unified Interface**: Single API for all authentication methods
- **Better Organization**: Clear separation of concerns
- **Easier Extension**: New strategies can be added without changing core protocol
- **Consistent Validation**: Unified validation context and process
- **Maintainability**: Cleaner, more maintainable code structure

## Migration Path

1. **Phase 1**: Implement core strategy interface
2. **Phase 2**: Migrate existing Mode W and Mode A to strategies
3. **Phase 3**: Add new strategies as needed
4. **Phase 4**: Deprecate old mode-based API

## Notes

- All strategies maintain backward compatibility with existing AIPs
- The unified API simplifies integration while maintaining flexibility
- Strategy pattern enables easy addition of new authentication methods
- Implementation details are left to concrete strategy classes

---
