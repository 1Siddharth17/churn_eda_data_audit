#  Monitoring Plan – Part 1: EDA & Data Quality
## Data Exploration and Quality Assurance Monitoring

## 1. Purpose

Part 1 (`eda_audit.ipynb`) performs **Exploratory Data Analysis and Data Quality Checks**. Monitoring ensures that:
-  Data quality remains consistent
-  No unexpected patterns emerge
-  Data is suitable for downstream analysis
-  Issues are caught early


## 2. What to Monitor

### 2.1 Data Completeness

**Track:**
- Missing values percentage by column
- Records with missing values
- Columns with > 10% missing data

**Alert if:**
```
CRITICAL:
- Any column > 30% missing values
- Customer ID column has any missing values
- Churn label column > 5% missing

WARNING:
- New columns with missing values
- Sudden increase in missing values
- Pattern change in missing data
```

### 2.2 Data Consistency

**Track:**
```
 Data types correct:
  - customer_id: string/int
  - dates: datetime
  - amounts: numeric
  - flags: boolean/int

 Value ranges valid:
  - recency_days: 0-730
  - monetary_value: positive
  - frequency: non-negative
  - engagement_score: 0-100
  - return_rate: 0-1
  - churn: 0 or 1
```

**Alert if:**
```
CRITICAL:
- Data type changes unexpectedly
- Negative values in monetary_value
- Churn values outside [0, 1]
- Future dates in recency
- Invalid customer IDs

WARNING:
- New value ranges detected
- Minimum value < previous minimum
- Maximum value > previous maximum
- New categories in categorical fields
```

### 2.3 Data Distribution (Exploratory)

**Track daily/weekly:**

```python
# Example distributions to monitor
distributions = {
    'recency_days': {
        'mean': ...,
        'median': ...,
        'std': ...,
        'min': ...,
        'max': ...,
        'skewness': ...
    },
    'frequency': { ... },
    'monetary_value': { ... },
    'engagement_score': { ... },
    'churn': {
        'rate': 0.25,  # % of churned customers
        'count': 1250
    }
}
```

**Alert if:**
```
WARNING:
- Churn rate shifts > 20%
- Mean of any feature changes > 15%
- Skewness indicates new outliers
- Unexpected distribution shape change
```

### 2.4 Outliers & Anomalies

**Track:**
- Count of outliers (IQR method)
- Extreme values (> 3 std from mean)
- Unusual combinations (e.g., high monetary, zero frequency)

**Alert if:**
```
WARNING:
- Outlier count increases > 50%
- New extreme values detected
- Sudden spike in anomalies

CRITICAL:
- Data appears corrupted
- All values the same (no variance)
- Invalid outlier patterns
```

### 2.5 Duplicate Records

**Track:**
```
- Count of exact duplicates
- Count of customer_id duplicates
- Duplicate ratio (% of data)
```

**Alert if:**
```
WARNING:
- Duplicate count increases
- Customer ID has duplicates
- New duplicate patterns

CRITICAL:
- > 5% of data is duplicate
- Primary key (customer_id) duplicated
```

---

## 3. Monitoring Frequency & Automation

```python
# Create script: monitoring/part1_weekly_audit.py

import pandas as pd
import numpy as np
from datetime import datetime

class Part1Monitor:
    def __init__(self, data_path):
        self.df = pd.read_csv(data_path)
        self.report = {}
    
    def check_completeness(self):
        """Check for missing values"""
        missing = self.df.isnull().sum()
        missing_pct = (missing / len(self.df)) * 100
        
        critical_issues = missing_pct[missing_pct > 30]
        warning_issues = missing_pct[(missing_pct > 10) & (missing_pct <= 30)]
        
        self.report['completeness'] = {
            'critical': critical_issues.to_dict(),
            'warning': warning_issues.to_dict(),
            'timestamp': datetime.utcnow().isoformat()
        }
        
        return len(critical_issues) == 0
    
    def check_consistency(self):
        """Check data types and value ranges"""
        issues = []
        
        # Check data types
        expected_types = {
            'customer_id': 'object',
            'recency_days': 'int64',
            'monetary_value': 'float64',
            'churn': 'int64'
        }
        
        for col, expected_type in expected_types.items():
            if col in self.df.columns and self.df[col].dtype != expected_type:
                issues.append(f"Type mismatch: {col} is {self.df[col].dtype}, expected {expected_type}")
        
        # Check value ranges
        if (self.df['monetary_value'] < 0).any():
            issues.append("Negative monetary values detected")
        
        if not self.df['churn'].isin([0, 1]).all():
            issues.append("Churn values outside [0, 1]")
        
        self.report['consistency'] = {
            'issues': issues,
            'timestamp': datetime.utcnow().isoformat()
        }
        
        return len(issues) == 0
    
    def check_duplicates(self):
        """Check for duplicate records"""
        exact_dups = self.df.duplicated().sum()
        id_dups = self.df.duplicated(subset=['customer_id']).sum()
        dup_ratio = (exact_dups / len(self.df)) * 100
        
        critical = dup_ratio > 5 or id_dups > 0
        
        self.report['duplicates'] = {
            'exact_duplicates': int(exact_dups),
            'id_duplicates': int(id_dups),
            'duplicate_ratio_pct': round(dup_ratio, 2),
            'critical': critical,
            'timestamp': datetime.utcnow().isoformat()
        }
        
        return not critical
    
    def check_distributions(self):
        """Track feature distributions"""
        distributions = {}
        
        numeric_cols = self.df.select_dtypes(include=[np.number]).columns
        
        for col in numeric_cols:
            distributions[col] = {
                'mean': round(self.df[col].mean(), 4),
                'median': round(self.df[col].median(), 4),
                'std': round(self.df[col].std(), 4),
                'min': round(self.df[col].min(), 4),
                'max': round(self.df[col].max(), 4),
                'skewness': round(self.df[col].skew(), 4)
            }
        
        # Check churn rate
        if 'churn' in self.df.columns:
            churn_rate = self.df['churn'].mean()
            distributions['churn_rate'] = round(churn_rate, 4)
        
        self.report['distributions'] = distributions
        return True
    
    def check_outliers(self):
        """Detect outliers using IQR method"""
        outlier_counts = {}
        
        numeric_cols = self.df.select_dtypes(include=[np.number]).columns
        
        for col in numeric_cols:
            Q1 = self.df[col].quantile(0.25)
            Q3 = self.df[col].quantile(0.75)
            IQR = Q3 - Q1
            
            lower_bound = Q1 - 1.5 * IQR
            upper_bound = Q3 + 1.5 * IQR
            
            outliers = ((self.df[col] < lower_bound) | (self.df[col] > upper_bound)).sum()
            outlier_counts[col] = int(outliers)
        
        self.report['outliers'] = outlier_counts
        return True
    
    def run_full_audit(self):
        """Run all checks"""
        results = {
            'completeness_ok': self.check_completeness(),
            'consistency_ok': self.check_consistency(),
            'duplicates_ok': self.check_duplicates(),
            'distributions_ok': self.check_distributions(),
            'outliers_ok': self.check_outliers(),
            'overall_ok': True,
            'timestamp': datetime.utcnow().isoformat()
        }
        
        results['overall_ok'] = all([
            results['completeness_ok'],
            results['consistency_ok'],
            results['duplicates_ok']
        ])
        
        return results, self.report

# Usage
monitor = Part1Monitor('data/customers.csv')
results, report = monitor.run_full_audit()

# Log results
import json
with open('logs/part1_audit.json', 'w') as f:
    json.dump(report, f, indent=2)

print(f"Audit Complete: {' PASS' if results['overall_ok'] else '❌ FAIL'}")
```

---

## 4. Alerting Rules

### Alert Levels


 CRITICAL (Immediate Action):
- > 30% missing values in any column
- Duplicate customer IDs
- Data type changes
- Negative monetary values
- Churn values outside [0, 1]
- > 5% duplicate records

 WARNING (Investigate):
- 10-30% missing values
- Churn rate shift > 20%
- Feature mean change > 15%
- Outlier count increase > 50%
- New anomalies detected

 INFO (Monitor):
- Routine distribution changes
- Minor data quality issues
- Expected seasonal variations
```


## 5. Weekly Checklist

**Every Monday:**
- [ ] Run Part 1 audit script
- [ ] Check for critical issues
- [ ] Review missing data patterns
- [ ] Verify data consistency
- [ ] Check for duplicate records
- [ ] Review distribution changes
- [ ] Log any anomalies
- [ ] Escalate if needed

**Monthly Review:**
- [ ] Analyze distribution trends
- [ ] Check for data drift patterns
- [ ] Review quality over time
- [ ] Plan any data cleaning
- [ ] Update baseline statistics


## 6. Retraining Triggers (for downstream models)

**Retrain Part 3 model if:**
```
DATA QUALITY TRIGGER:
- > 3 critical issues in audit
- New columns with data quality problems
- Significant shift in feature distributions
- Churn rate changes > 25%

ACTION:
- Re-run Part 1 audit
- Analyze root cause
- Clean data as needed
- Plan model retraining
```


## 7. Documentation Requirements

**Log Files to Keep:**
```
logs/
├── part1_audit.json          (weekly)
├── part1_quality_report.csv  (weekly)
├── part1_distributions.json  (monthly)
└── part1_issues.log          (ongoing)
```

## 8. Summary

**Monitor Daily:** Data quality issues in logs
**Review Weekly:** Run automated audit
**Review Monthly:** Analyze trends
**Alert When:** Critical thresholds exceeded

---


