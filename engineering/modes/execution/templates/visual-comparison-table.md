# Visual Comparison Table Template

Use this table format when comparing Figma designs with browser implementations.

## Comparison Table

| Check | Figma Reference | Implementation | Tolerance | Status |
|-------|-----------------|----------------|-----------|--------|
| Overall layout | Screenshot match | Browser render | Visual match | ✅/❌ |
| Component tree structure | Figma tree | DOM structure | Exact nesting | ✅/❌ |
| Spacing (gap, padding) | Layout Properties | Computed styles | ±2px | ✅/❌ |
| Colors (bg, text, border) | Token Mapping | Applied classes | Exact token | ✅/❌ |
| Typography (font, size, weight) | Token Mapping | Applied classes | Exact token | ✅/❌ |
| Component reuse | Component Instances | Imports used | All reused | ✅/❌ |
| Interactive states | States table | Browser behavior | Visual match | ✅/❌ |
| Border radius | Layout Properties | Computed styles | ±1px | ✅/❌ |
| Shadows/elevation | Effects list | CSS box-shadow | Visual match | ✅/❌ |

## Tolerance Guidelines

| Property | Acceptable Variance | Action if Exceeded |
|----------|--------------------|--------------------|
| Spacing | ±2px | Round to nearest token value |
| Colors | Exact match only | Use mapped token or flag deviation |
| Typography | Exact match only | Use mapped token or flag deviation |
| Border radius | ±1px | Round to nearest token value |
| Shadows | Visual approximation | Document in deviation notes |

## Status Indicators

- ✅ **Pass** – Implementation matches Figma within tolerance
- ⚠️ **Warning** – Minor deviation, documented but acceptable
- ❌ **Fail** – Significant deviation, requires fix or design review
