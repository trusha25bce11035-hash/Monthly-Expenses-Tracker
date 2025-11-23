"""
MONTHLY EXPENSES TRACKER
Complete expense management system with CRUD, predictions, and visualization
"""

import json
import os
import uuid
import hashlib
import secrets
import hmac
from datetime import datetime, timedelta
from collections import defaultdict

class AuthenticationManager:
    def __init__(self):
        self.users = {
            'admin': self._hash_password('admin123'),
            'user': self._hash_password('password123')
        }
        self.failed_attempts = {}
    
    def _hash_password(self, password: str) -> str:
        salt = secrets.token_hex(8)
        password_hash = hashlib.pbkdf2_hmac(
            'sha256', password.encode(), salt.encode(), 100000
        ).hex()
        return f"{salt}${password_hash}"
    
    def _verify_password(self, password: str, stored_hash: str) -> bool:
        try:
            salt, expected_hash = stored_hash.split('$', 1)
            new_hash = hashlib.pbkdf2_hmac(
                'sha256', password.encode(), salt.encode(), 100000
            ).hex()
            return hmac.compare_digest(f"{salt}${new_hash}", stored_hash)
        except:
            return False
    
    def authenticate(self, username: str, password: str) -> bool:
        if self.failed_attempts.get(username, 0) >= 3:
            print(" Account locked due to too many failed attempts.")
            return False
        
        if username in self.users and self._verify_password(password, self.users[username]):
            self.failed_attempts.pop(username, None)
            return True
        
        self.failed_attempts[username] = self.failed_attempts.get(username, 0) + 1
        remaining = 3 - self.failed_attempts[username]
        print(f"Invalid credentials. {remaining} attempts remaining.")
        return False

class ExpenseManager:
    def __init__(self):
        self.data_file = "expenses_data.json"
        self.expenses = self._load_data()
    
    def _load_data(self):
        """Load expenses data from file, create file if it doesn't exist"""
        try:
            if os.path.exists(self.data_file):
                with open(self.data_file, 'r') as f:
                    data = json.load(f)
                    print(f" Loaded {len(data)} expenses from file")
                    return data
            else:
                print(" Creating new expenses file...")
                with open(self.data_file, 'w') as f:
                    json.dump({}, f)
                return {}
        except Exception as e:
            print(f"Error loading data: {e}")
            return {}
    
    def _save_data(self):
        """Save expenses data to file"""
        try:
            with open(self.data_file, 'w') as f:
                json.dump(self.expenses, f, indent=2)
            print(f" Saved {len(self.expenses)} expenses to file")
        except Exception as e:
            print(f"Error saving data: {e}")
    
    def create_expense(self, user_id: str, amount: float, category: str, description: str, date: str) -> str:
        expense_id = str(uuid.uuid4())[:8]
        expense_data = {
            'id': expense_id,
            'user_id': user_id,
            'amount': float(amount),
            'category': category,
            'description': description,
            'date': date,
            'created_at': datetime.now().isoformat()
        }
        
        self.expenses[expense_id] = expense_data
        self._save_data()
        print(f" Created expense {expense_id} for user {user_id}")
        return expense_id
    
    def get_user_expenses(self, user_id: str):
        """Get all expenses for a specific user"""
        user_expenses = []
        for expense_id, expense in self.expenses.items():
            if expense.get('user_id') == user_id:
                user_expenses.append(expense)
        
        print(f" Found {len(user_expenses)} expenses for user {user_id}")
        return user_expenses
    
    def update_expense(self, expense_id: str, updates: dict) -> bool:
        if expense_id in self.expenses:
            self.expenses[expense_id].update(updates)
            self._save_data()
            print(f" Updated expense {expense_id}")
            return True
        print(f" Expense {expense_id} not found for update")
        return False
    
    def delete_expense(self, expense_id: str) -> bool:
        if expense_id in self.expenses:
            del self.expenses[expense_id]
            self._save_data()
            print(f" Deleted expense {expense_id}")
            return True
        print(f" Expense {expense_id} not found for deletion")
        return False

class ExpensePredictor:
    def predict_future_expenses(self, expenses: list, months: int = 3) -> dict:
        if len(expenses) < 2:
            print(" Need at least 2 expenses for predictions")
            return {}
        
        # Sort by date
        sorted_expenses = sorted(expenses, key=lambda x: x['date'])
        amounts = [exp['amount'] for exp in sorted_expenses]
        
        print(f" Analyzing {len(amounts)} expenses for prediction...")
        
        # Calculate average monthly growth
        if len(amounts) >= 2:
            growth_rate = (amounts[-1] - amounts[0]) / max(len(amounts), 1)
            if amounts[0] != 0:
                growth_rate_percent = growth_rate / amounts[0]
            else:
                growth_rate_percent = 0.05
        else:
            growth_rate_percent = 0.05  # Default 5% growth
        
        predictions = {}
        try:
            last_date = datetime.strptime(sorted_expenses[-1]['date'], '%Y-%m-%d')
        except:
            last_date = datetime.now()
        
        current_avg = sum(amounts) / len(amounts)
        
        for i in range(1, months + 1):
            future_date = last_date + timedelta(days=30 * i)
            month_key = future_date.strftime('%Y-%m')
            
            # Predict amount with growth rate
            predicted_amount = current_avg * (1 + growth_rate_percent * i)
            confidence = max(60, 90 - (i * 10))  # Decrease confidence for further months
            
            predictions[month_key] = {
                'predicted_amount': round(float(predicted_amount), 2),
                'confidence': confidence
            }
        
        print(f" Generated predictions for {months} months")
        return predictions

class ExpenseVisualizer:
    def show_category_distribution(self, expenses: list):
        if not expenses:
            print(" No expenses to display.")
            return
        
        category_totals = defaultdict(float)
        for exp in expenses:
            category_totals[exp['category']] += exp['amount']
        
        total = sum(category_totals.values())
        print("\n EXPENSE CATEGORY DISTRIBUTION")
        print("=" * 50)
        
        for category, amount in sorted(category_totals.items(), key=lambda x: x[1], reverse=True):
            percentage = (amount / total) * 100
            bar_length = int(percentage / 2)  # Each █ represents 2%
            bar = "█" * bar_length
            print(f"{category:<15} ${amount:>8.2f} ({percentage:>5.1f}%) {bar}")
        
        print(f"\n Total Expenses: ${total:.2f}")
    
    def show_monthly_summary(self, expenses: list):
        if not expenses:
            print(" No expenses to analyze.")
            return
        
        monthly_totals = defaultdict(float)
        for exp in expenses:
            year_month = exp['date'][:7]  # Get YYYY-MM
            monthly_totals[year_month] += exp['amount']
        
        print("\n MONTHLY EXPENSE SUMMARY")
        print("=" * 40)
        for month in sorted(monthly_totals.keys()):
            print(f"{month}: ${monthly_totals[month]:.2f}")
    
    def show_expense_statistics(self, expenses: list):
        if not expenses:
            print(" No expense data available.")
            return
        
        amounts = [exp['amount'] for exp in expenses]
        total = sum(amounts)
        avg = total / len(amounts)
        highest = max(amounts)
        lowest = min(amounts)
        
        print("\n EXPENSE STATISTICS")
        print("=" * 35)
        print(f"Total Expenses:    ${total:>10.2f}")
        print(f"Average Expense:   ${avg:>10.2f}")
        print(f"Highest Expense:   ${highest:>10.2f}")
        print(f"Lowest Expense:    ${lowest:>10.2f}")
        print(f"Number of Expenses: {len(expenses):>10}")

class MonthlyExpensesApp:
    def __init__(self):
        self.auth = AuthenticationManager()
        self.expense_manager = ExpenseManager()
        self.predictor = ExpensePredictor()
        self.visualizer = ExpenseVisualizer()
        self.current_user = None
        self.categories = [
            'Food', 'Transport', 'Entertainment', 
            'Utilities', 'Healthcare', 'Shopping', 'Other'
        ]
    
    def run(self):
        print("=" * 50)
        print(" MONTHLY EXPENSES TRACKER")
        print("=" * 50)
        
        if not self.login():
            return
        
        while True:
            self.show_main_menu()
            choice = input("\nEnter your choice (1-8): ").strip()
            
            if choice == '1':
                self.add_expense()
            elif choice == '2':
                self.view_expenses()
            elif choice == '3':
                self.update_expense()
            elif choice == '4':
                self.delete_expense()
            elif choice == '5':
                self.predict_expenses()
            elif choice == '6':
                self.visualize_data()
            elif choice == '7':
                self.show_statistics()
            elif choice == '8':
                print("\n Thank you for using Expense Tracker! Goodbye!")
                break
            else:
                print(" Invalid choice. Please try again.")
    
    def login(self):
        print("\n LOGIN")
        print("-" * 20)
        
        for attempt in range(3):
            username = input("Username: ").strip()
            password = input("Password: ").strip()
            
            if self.auth.authenticate(username, password):
                self.current_user = username
                print(f"\n Welcome back, {username}!")
                return True
            
            if attempt < 2:
                print(f"Please try again. {2 - attempt} attempts left.\n")
        
        print("\nLogin failed. Exiting application.")
        return False
    
    def show_main_menu(self):
        print("\n" + "=" * 40)
        print(" MAIN MENU")
        print("=" * 40)
        print("1.  Add New Expense")
        print("2.  View All Expenses")
        print("3.   Update Expense")
        print("4.   Delete Expense")
        print("5.  Predict Future Expenses")
        print("6.  Visualize Expenses")
        print("7.  View Statistics")
        print("8.  Exit")
        print("=" * 40)
    
    def add_expense(self):
        print("\n➕ ADD NEW EXPENSE")
        print("-" * 20)
        
        try:
            amount = float(input("Enter amount: $"))
            if amount <= 0:
                print(" Amount must be positive.")
                return
            
            print("\nAvailable categories:")
            for i, category in enumerate(self.categories, 1):
                print(f"  {i}. {category}")
            
            cat_choice = input("\nSelect category (number or name): ").strip()
            if cat_choice.isdigit() and 1 <= int(cat_choice) <= len(self.categories):
                category = self.categories[int(cat_choice) - 1]
            elif cat_choice in self.categories:
                category = cat_choice
            else:
                category = 'Other'
                print(" Using 'Other' category.")
            
            description = input("Description: ").strip() or "No description"
            
            date_input = input("Date (YYYY-MM-DD) or Enter for today: ").strip()
            if not date_input:
                date = datetime.now().strftime("%Y-%m-%d")
            else:
                # Validate date format
                datetime.strptime(date_input, '%Y-%m-%d')
                date = date_input
            
            expense_id = self.expense_manager.create_expense(
                self.current_user, amount, category, description, date
            )
            
            print(f"\n Expense added successfully!")
            print(f" ID: {expense_id}, Amount: ${amount:.2f}, Category: {category}")
            
        except ValueError as e:
            print(f" Invalid input: {e}")
        except Exception as e:
            print(f" Error adding expense: {e}")
    
    def view_expenses(self):
        expenses = self.expense_manager.get_user_expenses(self.current_user)
        
        if not expenses:
            print("\n No expenses found. Add some expenses first!")
            return
        
        print(f"\n YOUR EXPENSES (Total: {len(expenses)})")
        print("=" * 70)
        print(f"{'ID':<10} {'Date':<12} {'Category':<15} {'Amount':<10} {'Description'}")
        print("-" * 70)
        
        total = 0
        for exp in sorted(expenses, key=lambda x: x['date'], reverse=True):
            print(f"{exp['id']:<10} {exp['date']:<12} {exp['category']:<15} ${exp['amount']:<9.2f} {exp['description']}")
            total += exp['amount']
        
        print("-" * 70)
        print(f"{'TOTAL':<37} ${total:.2f}")
    
    def update_expense(self):
        expenses = self.expense_manager.get_user_expenses(self.current_user)
        if not expenses:
            print("\n No expenses to update. Add some expenses first!")
            return
        
        self.view_expenses()
        expense_id = input("\nEnter expense ID to update: ").strip()
        
        # Find the expense and verify ownership
        expense_to_update = None
        for exp in expenses:
            if exp['id'] == expense_id:
                expense_to_update = exp
                break
        
        if not expense_to_update:
            print(" Expense not found or you don't have permission to update it.")
            return
        
        print(f"\nUpdating Expense: {expense_id}")
        print(f"Current amount: ${expense_to_update['amount']:.2f}")
        print(f"Current category: {expense_to_update['category']}")
        print(f"Current description: {expense_to_update['description']}")
        
        updates = {}
        
        new_amount = input("\nNew amount (press Enter to keep current): ").strip()
        if new_amount:
            try:
                updates['amount'] = float(new_amount)
                if updates['amount'] <= 0:
                    print(" Amount must be positive.")
                    return
            except ValueError:
                print(" Invalid amount.")
                return
        
        new_category = input("New category (press Enter to keep current): ").strip()
        if new_category:
            if new_category in self.categories:
                updates['category'] = new_category
            else:
                print("  Category not recognized. Keeping current category.")
        
        new_description = input("New description (press Enter to keep current): ").strip()
        if new_description:
            updates['description'] = new_description
        
        if updates:
            if self.expense_manager.update_expense(expense_id, updates):
                print(" Expense updated successfully!")
            else:
                print(" Failed to update expense.")
        else:
            print("ℹ  No changes made.")
    
    def delete_expense(self):
        expenses = self.expense_manager.get_user_expenses(self.current_user)
        if not expenses:
            print("\n No expenses to delete. Add some expenses first!")
            return
        
        self.view_expenses()
        expense_id = input("\nEnter expense ID to delete: ").strip()
        
        # Verify expense exists and belongs to user
        user_expense_ids = [exp['id'] for exp in expenses]
        if expense_id not in user_expense_ids:
            print(" Expense not found or you don't have permission to delete it.")
            return
        
        confirm = input("Are you sure you want to delete this expense? (y/N): ").strip().lower()
        if confirm == 'y':
            if self.expense_manager.delete_expense(expense_id):
                print(" Expense deleted successfully!")
            else:
                print(" Failed to delete expense.")
        else:
            print("ℹ  Deletion cancelled.")
    
    def predict_expenses(self):
        expenses = self.expense_manager.get_user_expenses(self.current_user)
        
        if len(expenses) < 2:
            print("\n Need at least 2 expenses for predictions. Add more expenses first!")
            return
        
        try:
            months = int(input("Enter number of months to predict (1-6): ").strip())
            months = max(1, min(6, months))  # Limit to 1-6 months
        except ValueError:
            months = 3
            print("  Using default: 3 months")
        
        predictions = self.predictor.predict_future_expenses(expenses, months)
        
        if predictions:
            print(f"\n EXPENSE PREDICTIONS (Next {months} months)")
            print("=" * 50)
            for month, prediction in predictions.items():
                print(f" {month}: ${prediction['predicted_amount']:.2f} "
                      f"(Confidence: {prediction['confidence']}%)")
        else:
            print(" Could not generate predictions.")
    
    def visualize_data(self):
        expenses = self.expense_manager.get_user_expenses(self.current_user)
        
        if not expenses:
            print("\n No expenses to visualize. Add some expenses first!")
            return
        
        print("\n DATA VISUALIZATION")
        print("-" * 25)
        print("1. Category Distribution")
        print("2. Monthly Summary")
        print("3. Back to Main Menu")
        
        choice = input("\nChoose option (1-3): ").strip()
        
        if choice == '1':
            self.visualizer.show_category_distribution(expenses)
        elif choice == '2':
            self.visualizer.show_monthly_summary(expenses)
        elif choice == '3':
            return
        else:
            print(" Invalid choice.")
    
    def show_statistics(self):
        expenses = self.expense_manager.get_user_expenses(self.current_user)
        self.visualizer.show_expense_statistics(expenses)

def main():
    """Main function to run the application"""
    try:
        app = MonthlyExpensesApp()
        app.run()
    except KeyboardInterrupt:
        print("\n\n Application interrupted. Goodbye!")
    except Exception as e:
        print(f"\n Unexpected error: {e}")

if __name__ == "__main__":
    main()
