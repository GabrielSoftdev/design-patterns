# DESIGN PATTERNS WITH JAVA

I built a project to exercise "Object Calisthenics" using some of the more common design patterns like Strategy, Chain of responsibility, Model Method, State, Command, Observer.
The project in question is based on some e-commerce rules, such as calculating discounts, budgets, taxes, etc.

## Strategy

The first problem encountered was the demand for many “if's”, as the rules and number of taxes grew:

```java

public enum TaxType {
    ISS, ICMS;
}

public class Budget {
    private BigDecimal value;
    private int itensQuantity;

    public Budget(BigDecimal value, int itensQuantity) {
        this.value = value;
        this.itensQuantity = itensQuantity;
    }

    public BigDecimal getValue() {
        return value;
    }

    public int getItensQuantity() {
        return itensQuantity;
    }
}

public class TaxesCalculator {

    public BigDecimal calculateBudget(Budget budget, TaxType taxType) {
        
        // Here's it the problem. that doesn't look good
        
        switch (taxType) {
            case ISS -> {
                return budget.getValor().multiply(new BigDecimal("0.6"));
            }
            case ICMS -> {
                return budget.getValor().multiply(new BigDecimal("0.1"));
            }
            default -> {
                return BigDecimal.ZERO;
            }

        }
    }

}

```


so to combat this inconsistency and lack of scalability, I made use of the Strategy pattern, which consists of delegating logics not to the method as shown above, full of "if's", but to the instances or variations of the logic, as they are similar, varying only their values that are constant:

```java

public class Budget {
    private BigDecimal value;
    private int itensQuantity;

    public Budget(BigDecimal value, int itensQuantity) {
        this.value = value;
        this.itensQuantity = itensQuantity;
    }

    public BigDecimal getValue() {
        return value;
    }

    public int getItensQuantity() {
        return itensQuantity;
    }
}

public interface Tax {
    public BigDecimal calcular(Budget budget);
}

public class ICMS implements Tax {
    public BigDecimal calcular(Budget budget) {
        return budget.getValor().multiply(new BigDecimal("0.06"));
    }
}

public class ISS implements Tax {
    public BigDecimal calcular(Budget budget) {
        return budget.getValor().multiply(new BigDecimal("0.01"));
    }
}

public class TaxesCalculator {

    // oh yes, much better.
    
    public BigDecimal calculateBudget(Budget budget, Tax tax) {
        return tax.calcular(budget);
    }

}

```