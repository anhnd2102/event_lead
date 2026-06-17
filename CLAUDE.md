# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Purpose

Data analysis project focused on finding actionable insights to improve course completion rates. The primary data sources are CSV files containing student performance data (attendance, homework, exam results) for two courses: **Khóa 34** and **Khóa 03**.

## Data Analysis Stack

- **Python 3.14** with `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `statsmodels`
- **Jupyter notebooks** (`.ipynb`) for exploratory analysis
- **Plotly / Chart.js** for interactive visualizations
- CSV as the primary data interchange format

## Data Source Files

All data files live in `docs/`. Always use these exact files:

| File | Course | Rows | Columns | Notes |
|------|--------|------|---------|-------|
| `docs/EVENT_LEAD - Khoi34-2025.csv` | Khóa 34 | 1,821 | 15 | |
| `docs/EVENT_LEAD - Khoi03-2025.csv` | Khóa 03 | 1,192 | 16 | test5, test6 are 95%+ null |

### Schema (both courses)

| Column | Type | Missing | Notes |
|--------|------|---------|-------|
| `id` | int64 | 0% | Student ID |
| `name` | str | 0% | Họ tên |
| `status` | str | 0% | See status values below |
| `attendance_rate` | float64 | ~6% | Tỷ lệ chuyên cần (0-1) |
| `homework_rate` | float64 | ~6% | Tỷ lệ làm bài tập (0-1) |
| `test1`-`test5` (K34) | float64 | 19-33% | Điểm kiểm tra |
| `test1`-`test6` (K03) | float64 | 13-95% | test5/test6: 95% null (ít dữ liệu) |
| `class_id` | int64 | 0% | |
| `class_name` | str | 0% | Mã lớp (e.g. IC1992) |
| `teacher_name` | str | 0% | Giáo viên |
| `location_name` | str | 0% | Cơ sở / Online |
| `Khóa` | str | 0% | "Khóa 34" or "Khóa 03" |

### Status values

- **Hoàn thành** — completed (K34: 1048, K03: 765)
- **Không hoàn thành** — incomplete (K34: 259, K03: 205)
- **Bỏ học** — dropped out (K34: 169, K03: 86)
- **Bảo lưu** — deferred (K34: 145, K03: 53)
- **transferred** — chuyển khóa (K34: 150, K03: 34)
- **pending** — đang chờ (K34: 20, K03: 36)
- **Đang học** — still studying (K34: 16, K03: 2)
- **Chuyển lớp** — changed class (K34: 14, K03: 11)

### Location types

- **Online** is the largest location (~47% K34, ~49% K03)
- Many physical campuses in Hanoi (Phan Văn Trường, Hoàng Cầu, Nghĩa Đô, etc.)
- Some location names appear with/without "Cơ sở" prefix — may need normalization

### Data quality notes

- **Missing data increases with later tests**: test1 (~13-19%) → test5 (~33-95%) — students who drop out stop taking tests
- **K03 test5/test6**: 95%+ null — these exams barely existed; exclude or treat separately
- **attendance_rate & homework_rate**: only ~6% null (plentiful)
- Locations with/without "Cơ sở" prefix may refer to the same place — check before grouping

## Common Analysis Patterns

### Load a dataset
```python
import pandas as pd
df = pd.read_csv("docs/EVENT_LEAD - Khoi34-2025.csv")
print(df.info())
print(df["status"].value_counts())
```

### Define completion success (target variable)
```python
df["completed"] = df["status"] == "Hoàn thành"
```

### Check features vs completion rate
```python
df.groupby("location_name")["completed"].mean().sort_values()
df.groupby("teacher_name")["completed"].mean().sort_values()
```

## Setup

```bash
# Activate virtual environment (pandas, plotly, scipy, matplotlib, seaborn pre-installed)
source ~/.venv/bin/activate

# Or use uv run (no activate needed)
uv run python3 script.py
```

## Repository Structure

```
Event_Leadkhoi/
├── CLAUDE.md                # This file
├── docs/
│   ├── EVENT_LEAD - Khoi34-2025.csv   # Khóa 34: 1,821 students, 15 cols
│   └── EVENT_LEAD - Khoi03-2025.csv   # Khóa 03: 1,192 students, 16 cols
└── .claude/
    └── agents/
        └── sparring-partner.md
```

## Output Format

**Every analysis must produce an HTML file** with:
- Biểu đồ so sánh trực quan (Chart.js, Plotly, hoặc matplotlib → HTML)
- Các insight được trình bày rõ ràng bằng tiếng Việt
- Có thể mở trực tiếp bằng trình duyệt

## Data Analysis Best Practices

- Always inspect CSV structure (`df.info()`, `df.describe()`, `df.head()`) before analysis
- Handle missing data explicitly — document why rows are dropped or imputed
- Use Vietnamese for chart labels, UI text, and analysis commentary
- Export key visualizations to HTML (Plotly/Chart.js) for sharing
- Keep analysis notebooks reproducible — avoid cell-state dependencies
- Prefer `pandas` for data wrangling, `scipy`/`statsmodels` for statistical tests
- When building predictive models, keep feature engineering separate from model training
- Document assumptions behind any segmentation or threshold choices
- Use absolute paths (`docs/...`) so scripts work from any directory
- Watch for near-duplicate location names when grouping by campus