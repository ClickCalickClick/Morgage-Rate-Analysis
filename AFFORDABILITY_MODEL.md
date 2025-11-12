# Affordability Model & Calculator Formulas

This document details the financial models and formulas for the affordability and mortgage calculators.

## 1. Affordability Model

The goal is to give users a clear, research-backed guideline on whether a potential home is affordable for them. We will use the widely accepted **28/36 rule**.

### The 28/36 Rule

This rule is a standard underwriting guideline used by many lenders.

*   **The "28" (Front-End Ratio)**: Your total housing costs (Principal, Interest, Taxes, and Insurance - **PITI**) should not exceed **28%** of your **gross monthly income**.
*   **The "36" (Back-End Ratio)**: Your total debt payments (PITI + all other monthly debt like car loans, student loans, credit card payments) should not exceed **36%** of your **gross monthly income**.

Since we will only ask for income and not other debts, we will focus on the **28% rule** as the primary affordability metric. We will, however, mention the 36% rule as a disclaimer for the user to consider their other debts.

### Implementation

1.  **Inputs from User**:
    *   `yearlyIncome`: User's gross annual income.
    *   `homePrice`: The price of the home.
    *   `downPayment`: The down payment amount.
    *   (We will also need estimates for property tax and homeowners insurance, which can be based on national/regional averages as a percentage of home value, e.g., 1.2% for tax, 0.5% for insurance).

2.  **Calculation**:
    *   `grossMonthlyIncome = yearlyIncome / 12`
    *   `maxHousingCost = grossMonthlyIncome * 0.28`
    *   `estimatedMonthlyPITI = calculateMonthlyPayment(...) + (homePrice * 0.012 / 12) + (homePrice * 0.005 / 12)`

3.  **Output to User**:
    *   Compare `estimatedMonthlyPITI` with `maxHousingCost`.
    *   **If `estimatedMonthlyPITI <= maxHousingCost`**: "✅ Affordable. Your estimated monthly housing cost is within the recommended 28% of your gross income."
    *   **If `estimatedMonthlyPITI > maxHousingCost`**: "⚠️ Potentially Unaffordable. Your estimated monthly housing cost exceeds the recommended 28% of your gross income. Lenders may see this as a higher risk."

## 2. Mortgage Calculator Formulas

### Monthly Payment (P&I)

We will use the standard loan amortization formula to calculate Principal & Interest (P&I).

**Formula:**
`M = P * [r(1+r)^n] / [(1+r)^n - 1]`

*   `M`: Monthly Payment
*   `P`: Principal loan amount (`homePrice - downPayment`)
*   `r`: Monthly interest rate (`annualRate / 100 / 12`)
*   `n`: Number of payments (`loanTermInYears * 12`)

**Function:** `calculateMonthlyPayment(principal, annualRate, termYears)`

### Total Cost of Loan

This shows the user how much they will pay over the life of the loan.

**Formula:**
`Total Cost = (M * n) + downPayment`

**Function:** `calculateTotalCost(monthlyPayment, termYears, downPayment)`

### Refinance Break-Even Point

This helps a user decide if refinancing is worth it.

**Formula:**
`Break-Even Point (in months) = Total Refinancing Costs / Monthly Savings`

*   `Total Refinancing Costs`: These are the closing costs of the new loan (e.g., origination fees, appraisal fees). We can estimate this as a percentage of the loan amount (e.g., 2-5%).
*   `Monthly Savings`: The difference between the old monthly payment and the new, lower monthly payment.

**Function:** `calculateBreakEvenPoint(refiCosts, oldMonthlyPayment, newMonthlyPayment)`
The output will be "It will take you X months to recoup the costs of refinancing."
