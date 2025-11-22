# Monthly-Expenses-Tracker
A comprehensive expense tracker with CRUD operations, ML predictions, and data visualization. Users can add, view, update, and delete expenses, get future expense forecasts, and analyze spending patterns through text-based charts and statistics—all with secure authentication and data persistence.

 **PROBLEM STATEMENT**
* Individuals lack effective tools for tracking and analyzing monthly expenses
* Manual expense management leads to poor financial visibility and planning
* Need for automated expense forecasting and spending pattern analysis

**OBJECTIVES**
* Develop user-friendly expense tracking system
* Implement secure CRUD operations for expense management
* Provide ML-based expense predictions
* Generate visual spending insights
* Ensure data security and reliability

**FUNCTIONAL REQUIREMENTS**
* User authentication and authorization
* Add, view, update, delete expenses
* Categorize expenses (Food, Transport, Utilities, etc.)
* Predict future expenses (3-month forecast)
* Generate expense reports and visualizations
* Data export/backup functionality

**NON-FUNCTIONAL REQUIREMENTS**
* Security: Password hashing, user data isolation
* Reliability: Data persistence, error handling
* Usability: Intuitive CLI interface
* Performance: Fast data retrieval and processing
* Maintainability: Modular code structure

**SYSTEM ARCHITECTURE**
```
User Interface → Application Layer → Business Logic → Data Storage
     (CLI)          (Main App)      (CRUD, ML, Viz)    (JSON Files)
```

**PROCESS FLOW**
```
Login → Main Menu → Select Operation → Process Request → Display Results
```

**OVERVIEW**
* Python-based expense management system
* Combines expense tracking with predictive analytics
* Text-based visualization for spending insights
* File-based data storage

**FEATURES**
* Secure user authentication
* Complete expense CRUD operations
* Expense categorization
* Future expense predictions
* Text-based charts and statistics
* Monthly spending analysis

**TECHNOLOGIES USED**
* Python 3.x (Standard Library only)
* JSON for data storage
* PBKDF2 for password hashing
* UUID for expense identification

**INSTALLATION & RUNNING**
```bash
# 1. Ensure Python 3.x is installed
python --version

# 2. Download the script file
# 3. Run the application
python expense_tracker.py
```

**TESTING INSTRUCTIONS**
1. Login with: admin/admin123 or user/password123
2. Add sample expenses across different categories
3. Test all CRUD operations
4. Generate predictions (need 2+ expenses)
5. View visualizations and statistics

**USE CASE DIAGRAM**
```
Actors: User
Use Cases: 
  - Login
  - Add Expense
  - View Expenses  
  - Update Expense
  - Delete Expense
  - Predict Expenses
  - View Reports
```

**CLASS DIAGRAM**
```
AuthenticationManager
ExpenseManager
ExpensePredictor
ExpenseVisualizer  
MonthlyExpensesApp (Main Controller)
```

**SEQUENCE DIAGRAM**
```
User → MonthlyExpensesApp → ExpenseManager → JSON File
User → MonthlyExpensesApp → ExpensePredictor → Predictions
User → MonthlyExpensesApp → ExpenseVisualizer → Reports
```

**STORAGE DESIGN**
* JSON-based document storage
* Each expense as separate document
* User-based data isolation
* Schema: {id, user_id, amount, category, description, date, created_at}

**ML COMPONENT DETAILS**

**Dataset Description**
* User's historical expense data
* Features: amount, category, date, description
* Time-series expense records

**Model Selection Rationale**
* Moving Average algorithm chosen for:
  - Simplicity and interpretability
  - Works well with small datasets
  - No external dependencies
  - Suitable for expense trend analysis

**Evaluation Methodology**
* Prediction confidence scoring
* Based on data quantity and recency
* Manual validation against actual spending patterns
* Progressive confidence decrease for long-term predictions

**KEY ALGORITHMS**
* Moving Average for expense predictions
* PBKDF2 for password security
* Text-based visualization algorithms
* Date-based sorting and filtering
