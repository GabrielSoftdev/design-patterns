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

## Chain of Responsibility

Continuing with the construction of the e-commerce business rule, I came across a situation similar to the previous one, but this time there was a real need for conditionals because there were no pre-established values immediately, a chain of verifications was needed. The separation of types of discounts into classes was not enough, because even so, we would pass the responsibility of verifying the type of discount to another and only class.

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

public class DiscountCalculator {
    public BigDecimal calculateDiscount(Budget budget) {
        if (budget.getItensQuantity() > 5)
            return budget.getValue().multiply(new BigDecimal("0.1"));
        if (budget.getValue().compareTo(new BigDecimal("500")) > 0)
            return budget.getValue().multiply(new BigDecimal("0.1"));
        return BigDecimal.ZERO;
    }
}

```

And to solve this problem I had to rely on the chain of responsibility pattern, I destroyed the "if's" in classes linked as links by a parent class, thus enabling a self-assessment loop of these classes, letting them decide if the proposed value (budget) applies to the class method (discount calculation).


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

public abstract class Discount {
    protected Discount nextDiscount;

    public Discount(Discount nextDiscount) {
        this.nextDiscount = nextDiscount;
    }

    public abstract BigDecimal calculate(Budget budget);
}

public class DiscountByTotal extends Discount {
    public DiscountByTotal(Discount nextDiscount) {
        super(nextDiscount);
    }

    public BigDecimal calculate(Budget budget) {
        if (budget.getValue().compareTo(new BigDecimal("500")) > 0)
            return budget.getValue().multiply(new BigDecimal("0.6"));

        return nextDiscount.calculate(budget);
    }
}

public class DiscountByQuantity extends Discount {
    public DiscountByQuantity(Discount nextDiscount) {
        super(nextDiscount);
    }

    public BigDecimal calculate(Budget budget) {
        if (budget.getItensQuantity() > 5)
            return budget.getValue().multiply(new BigDecimal("0.1"));

        return nextDiscount.calculate(budget);
    }
}

// the WithoutDiscount class only serves to indicate the end of the search for discounts and apply the full amount of the budget like as an "else" or "default"
public class WithoutDiscount extends Discount{

    public WithoutDiscount() {
        super(null);
    }

    @Override
    public BigDecimal calculate(Budget budget) {
        return BigDecimal.ZERO;
    }
}

public class DiscountCalculator {
    public BigDecimal calculateDiscount(Budget budget) {
        Discount discount = new DiscountByTotal(new DiscountByQuantity(new WithoutDiscount()));
        return discount.calculate(budget);
    }
}
```
