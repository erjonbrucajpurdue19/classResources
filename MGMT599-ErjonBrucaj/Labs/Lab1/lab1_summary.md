# Lab 1 Summary

## Environment Setup
- **GCP Project ID**: mgmt599-ebrucaj-lab1
- **GitHub Repository**: https://github.com/erjonbrucajpurdue19/classResources
- **Data Lake Bucket**: mgmt599-ebrucaj-data-lake

## Key Findings
1. **Discounts above 30% consistently result in negative profits and margins**, especially at 50% and 80% discount levels.
2. **High-discount orders spike in November and December**, suggesting they may be linked to seasonal promotions like Black Friday.
3. **Some loyal customers consistently generate losses**, despite repeat purchases and moderate average discounts (20–35%).

## DIVE Analysis Results
- **Business Question Investigated**: What is the relationship between discounts and profitability?
- **Main Discovery**: High discounts (≥30%) lead to financial losses, and some repeat customers drive unprofitability due to consistent discount usage.
- **Strategic Recommendation**: Redesign the loyalty strategy to reward profitable behaviors, not just repeat purchases. Introduce profitability-based tiers and predictive models to reduce future losses.

## Challenges and Solutions
- **Challenge faced**: Interpreting why extreme losses were happening at specific discount levels and across different customer types.
- **How I solved it**: Used Gemini prompts to validate assumptions, segment customer behavior, and generate strategic insights. I also grouped discount tiers, validated patterns across months, and identified loss-making repeat customers through Python analysis.

## Time Spent
- **Environment setup**: 30 minutes  
- **Data lake creation**: 30 minutes  
- **Analysis**: 2.5 hours  
- **Total**: 4 hours
