---
title: "Developing an AI Intern for C-Level Executives with AWS Bedrock"
seoTitle: "AI Interns for Execs Using AWS Bedrock"
seoDescription: "Develop AI interns for executives with AWS Bedrock to enhance decision-making through AI-driven insights and strategic tools"
datePublished: Thu Nov 14 2024 17:59:21 GMT+0000 (Coordinated Universal Time)
cuid: cm3hm71ap000108mlcyb78p6e
slug: developing-an-ai-intern-for-c-level-executives-with-aws-bedrock
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1731607060703/bf0a01ac-abda-40f5-a2ba-4248e5f32768.jpeg
tags: ai, aws, technology, automation, culture, leadership, generative-ai

---

# Introduction

The journey to implementing AI Agents often seems straightforward in theory but proves complex in practice. This complexity stems from a fundamental challenge: bridging the gap between business leaders and technical teams. An organization's success with AI initiatives often correlates directly with how well these two groups communicate and share a common language - what we in the industry call Domain-Driven Design.

"But where are my AI Agents?" you might ask. Before diving into the technology, we must address the cultural foundation for success. Daily collaboration between business and IT teams is crucial for any AI Agent implementation.

You may wonder why invest time in yet another hyped technology. In this article, I'll demonstrate what a simple AI Agent can do for an executive through a practical example. This story builds on my previous article, [What C-Level Leaders Need to Know About AI Agents](https://blog.javatask.dev/what-c-level-leaders-need-to-know-about-ai-agents), where we introduced a CEO of a medium-sized bakery network who discovered that AI Agents function like interns with specific tools and capabilities.

> Note: While this article may appear lengthy, most of the content consists of detailed appendices and a sample AI-generated report for reference.

# The Story

After a sleepless night, our CEO (let's call him Ken) received a last-minute invitation to a hospitality conference. This sparked an idea: he needed better insights into his company's financial relationship with hotels. Given the time constraint, Ken asked his IT director and accountant to stay late and prepare an analysis of hotel billings and bookings for the first half of the year. Recognizing the opportunity, Ken shared his vision for AI Agents with his colleagues, mainly because they would spend several days preparing this ad-hoc report.

# The Result - AI Agent Generated Report

Generate report from a CEO request to AI Agent: “How did we do with the hospitality business in the first half of the year? Please send the report to email.”

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731605264271/877af470-4297-4fcc-a3c7-87dc474c2290.png align="center")

## \[Start\]Hospitality Business Performance Report - H1 2024

## Executive Summary

The hospitality business showed strong performance in the first half of 2024, with total bookings of $282,608 and total billings of $255,310. The average billing rate was $90.28, indicating a healthy revenue per booking.

## Detailed Analysis

### Overall Performance

* Total Bookings: $282,607.99
    
* Total Billings: $255,309.52
    
* Average Billing Rate: $90.28
    

### Monthly Trends

| Month | Bookings | Billings |
| --- | --- | --- |
| January | $47,376.71 | $44,865.32 |
| February | $48,402.91 | $42,915.77 |
| March | $46,599.87 | $41,660.99 |
| April | $47,852.04 | $44,383.40 |
| May | $48,651.34 | $43,993.93 |
| June | $43,725.12 | $37,490.11 |

### Key Observations

* Bookings peaked in May at $48,651.34, while billings were highest in January at $44,865.32.
    
* June showed a noticeable decline in both bookings and billings, which may require further investigation.
    
* The gap between bookings and billings suggests potential areas for improving conversion rates.
    

## Recommendations

1. Investigate the factors behind the June decline to mitigate any potential issues.
    
2. Analyze the discrepancy between bookings and billings to improve conversion rates.
    
3. Consider implementing strategies to maintain the strong performance seen in May throughout the summer months.
    

## \[End\]Conclusion

The hospitality business demonstrated a robust performance in the first half of 2024, with consistent bookings and billings. However, the June decline and the gap between bookings and billings present opportunities for improvement. Addressing these areas could lead to even stronger performance in the second half of the year.

## Technicalities

> Yes, in the article that has C-level in the title will be code and AWS Console, this is a reality that we live in ;)

### Understanding the Theory

If you've used ChatGPT for financial analysis, you've likely seen messages about generating and executing Python code. This represents a basic AI Agent with programming capabilities. While Large Language Models (LLMs) cannot perform mathematical calculations directly, they excel at generating code to perform these calculations and explaining the results.

We'll follow this approach for our CEO's Ad-hoc Financial Report Builder. Notice the specific naming convention—it's not just a generic "Report Builder" but "CEO's Financial Report Builder." This specificity reflects the tool's purpose: an intern with CEO-level data access, email authority, and strategic focus. This naming exemplifies the collaboration between business and IT teams.

### Data Flow

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731603600324/09827fee-cf1e-4701-87e7-a5c24a378dd6.png align="center")

The typical process includes:

1. Querying raw ERP data through SQL
    
2. Calculating industry segment metrics
    
3. Identifying trends
    
4. Generating formatted HTML reports
    
5. Sending email summaries
    

### Implementing the AI Intern

#### Prerequisites

* AWS Account with enabled LLM models (we'll use Anthropic Claude Sonnet v1)
    
* Clear instructions for the AI Intern
    
* Defined data access and processing tools
    

#### Tool Configuration

The AI Intern needs access to specific functions:

* `send_email`: Handles communication with preset CEO email settings
    
* `analyze_industry_performance`: Processes ERP data consistently
    

The entire Instruction text I attached in Appendix 1. Instruction to AI Intern starts like this:

```plaintext
You are an experienced financial analyst specializing in business finance. 
Your task is to prepare ad-hoc reports for the CEO based on the financial data 
provided and the specific request made. 
Follow these instructions carefully to produce a comprehensive 
and insightful report.

...
```

UI to create AI Intern looks like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731604317026/158d6762-f0bf-4897-8986-f1224a8c9825.png align="center")

Here you need to select Brain \[LLM Model\], provide Instructions and define Action Groups = Tools available to Intern.

For example, for our `send_email` function, the AI Intern needs to provide `subject` and `html_body` of email, the CEO’s email is already part of this function.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731604509625/697e9043-dbfc-45a0-bdea-d3b1dafa6bae.png align="center")

In my example, the tool is a Python function that is executed in a serverless manner; yes, you don’t need any server to run them. They are responsible for quering and filtering the data.

Python code for `analyze_industry_performance` function is:

```python
def analyze_industry_performance(df, industry=None, date_range=None):
    """
    Analyze financial performance by industry.
    Args:
        df: DataFrame with bakery financial data
        industry: Specific industry to analyze (optional)
        date_range: Tuple of (start_date, end_date) for analysis period
    """
    if date_range:
        df = df[(df['date'] >= date_range[0]) & (df['date'] <= date_range[1])]
   
    if industry:
        df = df[df['industry'] == industry]
    
    analysis = {
        'total_bookings': df['bookings'].sum(),
        'total_billings': df['billings'].sum(),
        'avg_billing_rate': df['billing_rate'].mean(),
        'industry_summary': df.groupby('industry').agg({
            'bookings': 'sum',
            'billings': 'sum',
            'billing_rate': 'mean'
        }).round(2),
        'monthly_trends': df.groupby('date').agg({
            'bookings': 'sum',
            'billings': 'sum'
        }).round(2)
    }
    
    return analysis
```

# Conclusion

While the setup might seem straightforward, successful implementation requires the following:

1. Reliable data infrastructure
    
2. Robust cybersecurity measures
    
3. Comprehensive API strategy
    
4. Expert oversight to validate AI-generated insights
    

The true challenge lies in the technology and establishing the necessary organizational foundation. Before embarking on an AI Agent initiative, consider the following:

* Is your data infrastructure ready?
    
* Do you have appropriate security measures in place?
    
* Have you developed a comprehensive API strategy?
    
* What investment is required?
    

AI Agents represent incredible potential, but their success depends on the proper infrastructure and culture. The technology is revolutionary - but only for organizations prepared to support it.

## Appendix 1. Full Instruction to Intern

```plaintext
You are an experienced financial analyst specializing in business finance. Your task is to prepare ad-hoc reports for the CEO based on the financial data provided and the specific request made. Follow these instructions carefully to produce a comprehensive and insightful report.

First, you will be presented with the financial data by using the function analyze_industry_performance
Next, you need to work on CEO's specific request.
If required, use the send_email function to send the HTML version of the report to the CEO's email.

Analyze the financial data in the context of the CEO's request. Consider the following steps:

1. Identify the key financial metrics relevant to the CEO's request.
2. Perform necessary calculations and comparisons.
3. Look for trends, patterns, or anomalies in the data.
4. Consider both short-term and long-term implications of the findings.

When preparing your report, adhere to these guidelines:

1. Be concise yet comprehensive.
2. Use clear, professional language.
3. Support your analysis with specific data points from the provided financial information.
4. Provide actionable insights and recommendations when appropriate.
5. Anticipate follow-up questions the CEO might have and address them proactively.

Present your report in the following format:

<report>
<executive_summary>
Provide a brief overview of the key findings and their significance (2-3 sentences).
</executive_summary>

<detailed_analysis>
Present your in-depth analysis, broken down into relevant subsections. Include supporting data, calculations, and reasoning for your conclusions.
</detailed_analysis>

<recommendations>
If applicable, offer strategic recommendations based on your analysis. Explain the potential impact of each recommendation.
</recommendations>

<conclusion>
Summarize the main points and reinforce the most critical insights for the CEO.
</conclusion>
</report>

Remember to tailor your analysis and report specifically to the CEO's request while leveraging your financial expertise to provide valuable insights.
```