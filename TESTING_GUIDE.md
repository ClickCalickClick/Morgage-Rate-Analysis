# Quick Test Guide - New Features

## Testing the Affordability Calculator

### Test Case 1: Affordable Home
1. Enter ZIP: `10001`
2. Home Price: `350,000`
3. Down Payment: `70,000` (20%)
4. Annual Income: `100,000`
5. Click "Analyze Rates"

**Expected**: Green ✅ Affordable card showing monthly housing cost within 28% guideline

### Test Case 2: Unaffordable Home
1. Enter ZIP: `10001`
2. Home Price: `800,000`
3. Down Payment: `80,000` (10%)
4. Annual Income: `80,000`
5. Click "Analyze Rates"

**Expected**: Yellow ⚠️ Potentially Unaffordable card with warning

### Test Case 3: No Income (Should show "Enter income to calculate")
1. Enter ZIP: `10001`
2. Leave Annual Income blank
3. Click "Analyze Rates"

**Expected**: Affordability card shows "Enter income to calculate affordability"

---

## Testing the Mortgage Calculator

### Test Case 1: Basic 30-Year Mortgage
1. Scroll to "Explore Your Mortgage"
2. Loan Amount: `240,000`
3. Interest Rate: `6.5`
4. Loan Term: `30 years`
5. Click "Calculate"

**Expected**: 
- Monthly Payment: ~$1,520
- Total Interest: ~$308,000
- Total Cost: ~$548,000

### Test Case 2: Slider Synchronization
1. Move "Loan Amount" slider
2. Verify the text input updates automatically
3. Change text input
4. Verify the slider updates automatically

**Expected**: Inputs and sliders always stay in sync

### Test Case 3: Rate Sensitivity
1. Loan Amount: `240,000`
2. Change Interest Rate from `6.5%` to `7.0%`
3. Observe monthly payment increase

**Expected**: Payment increases with higher rate

---

## Testing the Refinance Break-Even Calculator

### Test Case 1: Worthwhile Refinance
1. Click "Refinance Break-Even" tab
2. Current Monthly Payment: `1,500`
3. New Monthly Payment: `1,200`
4. Refinancing Costs: `5,000`
5. Click "Calculate Break-Even"

**Expected**: "You will break even in 20 months (1.7 years)"

### Test Case 2: Not Worth Refinancing
1. Current Monthly Payment: `1,500`
2. New Monthly Payment: `1,400`
3. Refinancing Costs: `15,000`
4. Click "Calculate Break-Even"

**Expected**: "You will break even in 150 months (12.5 years)"

### Test Case 3: No Savings
1. Current Monthly Payment: `1,500`
2. New Monthly Payment: `1,500`
3. Refinancing Costs: `5,000`
4. Click "Calculate Break-Even"

**Expected**: Error message about no savings

---

## Testing Down Payment Toggle

### Test Case 1: Dollar to Percent
1. Down Payment Type: Select "$"
2. Enter: `80000`
3. Home Price: `400000`
4. Switch Down Payment Type to "%"

**Expected**: Value converts to `20` (percent)

### Test Case 2: Percent to Dollar
1. Down Payment Type: Select "%"
2. Enter: `25`
3. Home Price: `400000`
4. Switch Down Payment Type to "$"

**Expected**: Value converts to `100000` (dollars)

### Test Case 3: Slider Updates
1. Move Down Payment slider
2. Verify both text input and slider update
3. Change between $ and %
4. Verify slider max changes (100 for %, home price for $)

**Expected**: All values stay synchronized

---

## Testing Navigation

### Test Case 1: Header Links
1. Click "Analyze" in header
2. Page scrolls to input panel
3. Click "Calculator"
4. Page scrolls to calculator section
5. Click "Charts"
6. Page scrolls to charts section

**Expected**: Smooth scrolling to each section

### Test Case 2: Settings Button
1. Click "⚙ Settings" button (or "⚙" on mobile)
2. Modal appears centered on screen
3. Click "Close" button
4. Modal disappears

**Expected**: Modal toggles open/closed correctly

---

## Testing Export/Import

### Test Case 1: Export Data
1. Click Settings button
2. Click "Export Data"
3. File downloads as `mortgage-rates-export-YYYY-MM-DD.json`
4. Open file and verify structure:
   - Contains `version`
   - Contains `exportedAt` timestamp
   - Contains `app` object with `lastZip`
   - Contains `rates` array

**Expected**: Valid JSON with proper schema

### Test Case 2: Import Data
1. Click Settings button
2. Select a previously exported JSON file
3. Choose "Merge" option
4. Verify message shows records imported

**Expected**: Import completes with success message

---

## Testing Lender Table

### Test Case 1: Table Display
1. Load national rates (automatic on startup)
2. Verify table shows:
   - Lender names
   - Interest rates
   - APR values
   - Calculated monthly payment
   - Product type
3. Check for striped row styling

**Expected**: Professional-looking table with all columns

### Test Case 2: Best Rate Card
1. Load any rate data
2. Verify "Best Rate Found" card shows:
   - Lowest rate percentage
   - Lender name offering best rate

**Expected**: Correct best rate identified and displayed

---

## Testing Mobile Responsiveness

### Test Case 1: Input Panel Mobile
1. View on mobile device (< 768px)
2. Verify:
   - 1 column layout
   - All inputs visible
   - "Analyze Rates" button full width

**Expected**: Single column, easy to tap

### Test Case 2: Dashboard Mobile
1. View summary cards and table
2. Verify:
   - Cards stack vertically
   - Table responsive/scrollable
   - Best rate card visible

**Expected**: Mobile-optimized layout

### Test Case 3: Header Mobile
1. View header on mobile
2. Verify:
   - Navigation hidden
   - Settings button (⚙) visible as icon
   - Title displays properly

**Expected**: Proper mobile header layout

### Test Case 4: Calculator Mobile
1. View calculator on mobile
2. Verify:
   - Inputs stack properly
   - Sliders responsive
   - Results display in stacked cards

**Expected**: Calculator usable on mobile

---

## Testing Calculation Accuracy

### Cross-Check with Online Calculators

**Standard mortgage payment formula**:
- Home Price: $300,000
- Down Payment: $60,000
- Loan Amount: $240,000
- Rate: 6.5%
- Term: 30 years
- Expected Monthly Payment: **~$1,520**

Compare with at least 2 online calculators to verify accuracy.

**Affordability calculation**:
- Income: $100,000 ($8,333/month)
- Max 28% housing cost: $2,333/month
- If PITI = $2,200 → Should show "Affordable"
- If PITI = $2,500 → Should show "Unaffordable"

---

## Known Limitations (to be fixed)

1. ⚠️ **Proxy URL is placeholder** - Must be updated with actual Netlify function
2. ⚠️ **No real rate data** - Will use national-averages.json until proxy is deployed
3. ⚠️ **Charts may not update** - Chart.js integration works but needs rate data
4. ⚠️ **Gemini API not integrated** - Works if key is provided, but requires setup
5. ⚠️ **No data persistence** - Dexie DB works but proxy needed for real data flow

---

## Browser DevTools Tips

### Check Console
1. Open DevTools (F12)
2. Go to Console tab
3. Check for any errors or warnings
4. Verify fetch calls are being attempted

### Check Network
1. Open DevTools Network tab
2. Click "Analyze Rates"
3. Watch for attempted fetch to proxy URL
4. Verify 404 or CORS error (expected with placeholder URL)

### Check Storage
1. Open DevTools Application tab
2. Check IndexedDB → MortgageRatesDB → rates
3. Verify records are being stored after import

---

## Success Criteria

✅ Affordability calculator shows correct status  
✅ Mortgage calculator shows accurate monthly payment  
✅ Refinance calculator shows correct break-even point  
✅ Down payment toggle works correctly  
✅ Table displays with all columns  
✅ Sliders synchronize with inputs  
✅ Export creates valid JSON file  
✅ Import works with valid JSON  
✅ Mobile layout is responsive  
✅ No console errors  

When all criteria are met, the implementation is complete! ✨
