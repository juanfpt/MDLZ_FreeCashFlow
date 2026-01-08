# Tax Classification Configuration

This file contains the classification rules for tax transactions in the Cash Taxes L Power Query script.

## File Location
The `TaxRules` Power Query expects the classification file at:
`Data Repository\Actuals\Support\Classifier.xlsx`

(Note: A copy may also exist in `Data Repository\Configuration\Tax_Classification.xlsx` for reference)

## Power Query Architecture

The classification system uses a **two-query architecture** to avoid Formula.Firewall errors:

### Query 1: TaxRules (reference-only)
- Loads classification rules from the Excel file
- Transforms column types and converts to uppercase
- Sorts by Priority (ascending)
- **Enable load:** ❌ DISABLED (reference-only query)

### Query 2: Cash Taxes L (main query)
- References `TaxRules` query with `Table.Buffer()` for performance
- Performs classification inline (no custom functions)
- Processes tax transaction files from the Cash Taxes folder
- **Enable load:** ✅ ENABLED

This architecture prevents cross-source data combination issues that trigger Formula.Firewall errors.

## Table Structure
The Excel file contains a table named `TaxClassification` with the following columns:

- **Country**: The country code(s) for which this rule applies
  - Single country: `CO`, `MX`, `PE`, etc.
  - Multiple countries: `CO,EC,PE` (comma-separated)
  - All countries: `ALL` (wildcard)

- **Keyword**: The text to search for in transaction descriptions (case-insensitive)

- **Category**: The classification category to assign when matched

- **Priority**: Numeric priority (lower number = higher priority)
  - Used to resolve conflicts when multiple keywords match
  - Within the same level (country-specific/multi-country/global)

## Classification Logic

The system uses a **3-level priority hierarchy**:

1. **Level 1: Country-specific rules** (highest priority)
   - Exact country match (e.g., `Country = "CO"`)
   
2. **Level 2: Multi-country rules**
   - Rules with multiple countries (e.g., `Country = "CO,EC,PE"`)
   
3. **Level 3: Global rules** (lowest priority)
   - Rules that apply to all countries (`Country = "ALL"`)

Within each level, rules are sorted by the `Priority` column (ascending order).

## Example Rules

| Country | Keyword | Category | Priority | Notes |
|---------|---------|----------|----------|-------|
| CO | MSA | WHT (MSA / Interest) | 1 | Colombia-specific |
| CO,EC,PE | Retención | WHT (MSA / Interest) | 1 | Multi-country rule |
| ALL | WHT | WHT (MSA / Interest) | 1 | Global fallback |
| CO,EC | Pago Autorretención P | Income Tax in advance | 0 | Higher priority than "Retención" |

## Classification Categories

Current categories include:
- WHT (MSA / Interest)
- Income Tax in advance
- Annual Income Tax
- Litigation
- Others (default when no match)

## How to Update

1. Open `Tax_Classification.xlsx` in Excel
2. Add/modify rows in the `TaxClassification` table
3. For overlapping keywords, adjust the `Priority` value
   - Lower numbers = higher priority
   - Example: "Pago Autorretención P" has priority 0 to match before "Retención" (priority 1)
4. Save the file
5. Refresh the Power Query query

## Important Notes

- Keywords are matched using case-insensitive substring search
- More specific keywords should have lower priority numbers
- If a keyword is a substring of another, use priority to control which matches first
- The system returns "Others" when no keyword matches
