# Test Coverage Analysis - JDBB Student Portal

## Current State

**Test Coverage: 0%**

The codebase currently has:
- ❌ No test files
- ❌ No testing framework configured
- ❌ No CI/CD test automation
- ❌ No package.json or test dependencies

The entire application is contained in a single `index.html` file (~880 lines) with embedded JavaScript.

---

## Critical Areas Requiring Test Coverage

### 1. **Price Calculation Logic** (HIGH PRIORITY) ⚠️

**Location:** `index.html:756-785` (calculateFees function) and `index.html:812-826` (submit form)

**Why Critical:**
- Directly affects student earnings and store revenue
- Complex tiered fee calculation (20%, 15%, 10% based on price)
- HST calculation (13%)
- Financial accuracy is essential

**Current Logic:**
```javascript
// Price tiers
preTaxPrice <= 250  → 20% fee
preTaxPrice <= 500  → 15% fee
preTaxPrice > 500   → 10% fee

// Calculation
preTaxPrice = priceFinal / 1.13
storeFee = preTaxPrice * feeRate
studentShare = preTaxPrice - storeFee
```

**Test Cases Needed:**
1. ✅ Verify HST calculation (13%) is accurate
2. ✅ Test tier boundary conditions ($250, $500)
3. ✅ Test edge cases (zero, negative, very large amounts)
4. ✅ Verify fee rates applied correctly per tier
5. ✅ Test rounding behavior (2 decimal places)
6. ✅ Test that studentShare + storeFee = preTaxPrice

**Example Test Scenarios:**
- Input: $113.00 → Pre-tax: $100 → Fee (20%): $20 → Student: $80
- Input: $282.50 → Pre-tax: $250 → Fee (20%): $50 → Student: $200
- Input: $282.51 → Pre-tax: $250.01 → Fee (15%): $37.50 → Student: $212.51
- Input: $565.00 → Pre-tax: $500 → Fee (15%): $75 → Student: $425
- Input: $565.01 → Pre-tax: $500.01 → Fee (10%): $50 → Student: $450.01

---

### 2. **Data Filtering and Aggregation** (HIGH PRIORITY)

**Location:** `index.html:665-712` (loadDashboard function)

**Why Critical:**
- Incorrect filtering could show wrong student data (privacy/security issue)
- Balance calculations affect payout amounts
- Stats drive student decision-making

**Current Logic:**
```javascript
myItems = items.filter(item => item.StudentID === currentStudent.id)
activeItems = myItems.filter(item => item.Status === 'Active' || item.Status === 'Pending')
soldItems = myItems.filter(item => item.Status === 'Sold')
balance = soldItems.reduce((sum, item) => sum + (parseFloat(item.StudentShare) || 0), 0)
```

**Test Cases Needed:**
1. ✅ Verify only current student's items are shown
2. ✅ Test status filtering (Active, Pending, Sold)
3. ✅ Verify balance calculation sums correctly
4. ✅ Test with empty data sets
5. ✅ Test with mixed statuses
6. ✅ Test parseFloat handling of null/undefined values
7. ✅ Test sorting by DateSubmitted

---

### 3. **Form Validation** (MEDIUM PRIORITY)

**Location:** `index.html:804-865` (submitForm handler)

**Why Important:**
- Prevents invalid data from entering the database
- Required fields must be validated
- Data types must be correct

**Current Validation:**
- HTML5 `required` attributes only
- No JavaScript validation beyond that

**Test Cases Needed:**
1. ✅ Test required fields (ItemName, Category, PriceFinal, Type)
2. ✅ Test numeric validation for prices
3. ✅ Test negative price rejection
4. ✅ Test very large prices
5. ✅ Test empty/whitespace-only inputs
6. ✅ Test special characters in text fields
7. ✅ Test consignment period validation

---

### 4. **Grist API Integration** (HIGH PRIORITY)

**Location:** Multiple functions using `docApi`

**Why Critical:**
- All data operations depend on correct API calls
- Errors could corrupt data or cause data loss
- Network errors need proper handling

**Current API Calls:**
- `docApi.fetchTable('Categories')` - line 648
- `docApi.fetchTable('StoreItems')` - line 667, 716
- `docApi.applyUserActions(['AddRecord', ...])` - line 846

**Test Cases Needed:**
1. ✅ Test successful data fetch
2. ✅ Test error handling for failed API calls
3. ✅ Test network timeout scenarios
4. ✅ Test malformed response handling
5. ✅ Test AddRecord with valid/invalid data
6. ✅ Mock Grist API responses
7. ✅ Test record creation with all field types

---

### 5. **UI State Management** (MEDIUM PRIORITY)

**Location:** Tab switching, form resets, message displays

**Why Important:**
- Poor state management creates confusing UX
- Forms should clear after submission
- Error messages should display/clear appropriately

**Current Functions:**
- `switchTab()` - line 787
- `showMainApp()` - line 635
- Form reset behavior - line 852

**Test Cases Needed:**
1. ✅ Test tab switching updates active states
2. ✅ Test form reset after successful submission
3. ✅ Test success/error message display
4. ✅ Test message auto-hide timeout
5. ✅ Test payout balance sync with dashboard balance
6. ✅ Test loading state management

---

### 6. **Authentication Flow** (HIGH PRIORITY)

**Location:** `index.html:615-633` (grist.onRecord handler)

**Why Critical:**
- Security: prevents unauthorized access
- Data isolation: ensures students only see their data

**Current Logic:**
```javascript
grist.onRecord(async function(record) {
    if (record && record.id) {
        currentStudent = {
            id: record.id,
            studentId: record.StudentID || record.id,
            name: record.PreferredName || record.LegalName || 'Student',
            // ...
        };
        showMainApp();
    }
});
```

**Test Cases Needed:**
1. ✅ Test with valid student record
2. ✅ Test with missing record
3. ✅ Test with partial student data
4. ✅ Test PreferredName fallback to LegalName
5. ✅ Test with undefined/null values
6. ✅ Test that loadingScreen hides when authenticated

---

### 7. **Edge Cases and Error Handling** (MEDIUM PRIORITY)

**Cross-cutting Concerns**

**Test Cases Needed:**
1. ✅ Test with empty categories list
2. ✅ Test with empty items list
3. ✅ Test date parsing/formatting
4. ✅ Test concurrent form submissions
5. ✅ Test browser back/forward navigation
6. ✅ Test with very long text inputs
7. ✅ Test special characters in item names/notes
8. ✅ Test XSS prevention in user inputs

---

## Recommended Testing Strategy

### Phase 1: Unit Tests (Foundation)

**Setup Required:**
1. Extract JavaScript into separate `.js` files
2. Install testing framework (Jest recommended)
3. Set up test environment with JSDOM for DOM testing

**Priority Functions to Extract & Test:**
```
src/
├── utils/
│   ├── priceCalculator.js    # calculateFees, fee tier logic
│   ├── dataFilters.js         # filtering/aggregation functions
│   └── validators.js          # form validation
├── api/
│   └── gristClient.js         # Grist API wrapper
└── __tests__/
    ├── priceCalculator.test.js
    ├── dataFilters.test.js
    ├── validators.test.js
    └── gristClient.test.js
```

**Estimated Coverage After Phase 1:** 60-70%

### Phase 2: Integration Tests

**Focus Areas:**
1. Form submission end-to-end flow
2. Data loading and display pipeline
3. Tab navigation with data refresh
4. Error recovery scenarios

**Tools:**
- Testing Library (for DOM interactions)
- MSW (Mock Service Worker) for API mocking

**Estimated Coverage After Phase 2:** 80-85%

### Phase 3: End-to-End Tests

**Critical User Journeys:**
1. Student logs in → views dashboard → submits item → sees updated stats
2. Student uses price calculator → submits item with calculated price
3. Student views items list → checks earnings
4. Student requests payout

**Tools:**
- Playwright or Cypress (Grist environment simulation)

**Estimated Coverage After Phase 3:** 90%+

---

## Specific Implementation Recommendations

### 1. Create Package Configuration

```json
{
  "name": "jdbb-student-portal",
  "version": "1.0.0",
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "jest-environment-jsdom": "^29.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/dom": "^9.0.0"
  }
}
```

### 2. Example Test File Structure

**tests/priceCalculator.test.js**
```javascript
describe('Price Calculator', () => {
  describe('Fee Tier Calculation', () => {
    test('applies 20% fee for pre-tax price ≤ $250', () => {
      const result = calculateStudentShare(113.00); // $100 pre-tax
      expect(result.studentShare).toBe(80.00);
      expect(result.feeRate).toBe(0.20);
    });

    test('applies 15% fee for pre-tax price $250-$500', () => {
      const result = calculateStudentShare(282.51); // $250.01 pre-tax
      expect(result.feeRate).toBe(0.15);
    });

    test('handles boundary at exactly $250', () => {
      const result = calculateStudentShare(282.50); // exactly $250 pre-tax
      expect(result.feeRate).toBe(0.20);
    });
  });

  describe('HST Calculation', () => {
    test('correctly removes 13% HST', () => {
      const result = calculateStudentShare(113.00);
      expect(result.preTaxPrice).toBeCloseTo(100.00, 2);
    });
  });
});
```

### 3. CI/CD Integration

Add GitHub Actions workflow:

```yaml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install
      - run: npm test
      - run: npm run test:coverage
```

---

## Priority Matrix

| Area | Priority | Risk Level | Complexity | Est. Tests |
|------|----------|------------|------------|------------|
| Price Calculation | 🔴 HIGH | Critical | Medium | 15-20 |
| Data Filtering | 🔴 HIGH | High | Low | 10-15 |
| Grist API Integration | 🔴 HIGH | High | High | 20-25 |
| Authentication | 🔴 HIGH | Critical | Medium | 8-10 |
| Form Validation | 🟡 MEDIUM | Medium | Low | 10-12 |
| UI State Management | 🟡 MEDIUM | Low | Medium | 12-15 |
| Edge Cases | 🟡 MEDIUM | Medium | Medium | 15-20 |

**Total Estimated Tests:** 90-117 test cases

---

## Immediate Action Items

1. **Week 1:** Set up testing infrastructure
   - Add package.json with Jest
   - Create test directory structure
   - Write first smoke test

2. **Week 2:** Implement critical path tests
   - Price calculator tests (20 tests)
   - Data filtering tests (15 tests)

3. **Week 3:** API and integration tests
   - Mock Grist API
   - Test data operations (25 tests)

4. **Week 4:** Complete coverage
   - Form validation tests
   - UI state tests
   - Edge cases

---

## Success Metrics

- **Code Coverage:** Target 85%+ line coverage
- **Critical Functions:** 100% coverage for price calculations
- **API Operations:** All Grist interactions mocked and tested
- **CI/CD:** All tests passing on every commit
- **Regression Prevention:** No bugs escape to production

---

## Conclusion

The JDBB Student Portal currently has **zero test coverage**, which represents a significant risk, especially for financial calculations and data integrity.

**Highest Priority:** Price calculation logic must be tested immediately, as errors could result in incorrect student payments.

**Recommended First Steps:**
1. Extract price calculation logic to separate file
2. Write comprehensive tests for all fee tiers
3. Set up CI/CD to run tests automatically
4. Gradually refactor remaining code for testability

**Estimated Effort:** 2-3 weeks to achieve 85% coverage with a structured approach.
