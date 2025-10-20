# Provided by Somayeh Komeylian: PhD Student at UCSD & SDSU #
# The provided Python code implements a simple banking application with a graphical user interface (GUI) using the tkinter library. 
# It includes a BankAccount class to manage account operations and a BankingApp class to handle the GUI.
# This is the banking account of Somayeh Komeylian with the recorded information of '12345': BankAccount(Full Name: "Somayeh Komeylian", Account Numeber: "12345", Her Balance: $1000),
# Somayeh Komeylian can also transfer money to another account with the recorded information of '67890': BankAccount("Ali Ahmadi", "67890", 500)
# You can change the whole of information based on your own names

import tkinter as tk
from tkinter import ttk, messagebox
import random

# --- BankAccount Class ---
class BankAccount:
    """
    Represents a simple bank account with deposit, withdrawal, and transfer capabilities.
    """
    def __init__(self, account_holder, account_number, balance=0):
        self.account_holder = account_holder
        self.account_number = account_number
        self.balance = balance

    def deposit(self, amount):
        """Deposits a specified amount into the account."""
        try:
            amount = float(amount)
            if amount > 0:
                self.balance += amount
                return True
            else:
                return False
        except ValueError:
            return False

    def withdraw(self, amount):
        """Withdraws a specified amount from the account."""
        try:
            amount = float(amount)
            if amount > 0 and self.balance >= amount:
                self.balance -= amount
                return True
            else:
                return False
        except ValueError:
            return False
            
    def transfer(self, recipient_account, amount):
        """Transfers a specified amount to another account."""
        try:
            amount = float(amount)
            if amount > 0:
                if self.withdraw(amount):  # Use the existing withdraw method
                    recipient_account.deposit(amount)  # Use the existing deposit method
                    return True
                else:
                    return False
            else:
                return False
        except ValueError:
            return False
        
# --- Tkinter GUI Application Class ---
class BankingApp:
    """
    The main GUI application for the banking system.
    """
    def __init__(self, master):
        self.master = master
        master.title("Banking Application")
        master.geometry("400x350")
        master.resizable(False, False)

        # In-memory storage for bank accounts
        self.accounts = {
            '12345': BankAccount("Somayeh Komeylian", "12345", 1000),
            '67890': BankAccount("Ali Ahmadi", "67890", 500)
        }
        
        self.current_account = None  # Stores the account of the logged-in user

        self.create_login_widgets()

    def create_login_widgets(self):
        """Creates the login screen widgets."""
        for widget in self.master.winfo_children():
            widget.destroy()

        self.login_frame = ttk.Frame(self.master, padding="20")
        self.login_frame.pack(expand=True)

        ttk.Label(self.login_frame, text="Log In", font=("Arial", 16, "bold")).pack(pady=10)

        ttk.Label(self.login_frame, text="Account Holder Name:").pack(anchor="w")
        self.name_entry = ttk.Entry(self.login_frame, width=30)
        self.name_entry.pack(pady=5)
        self.name_entry.focus()

        ttk.Label(self.login_frame, text="Account Number:").pack(anchor="w")
        self.account_number_entry = ttk.Entry(self.login_frame, width=30)
        self.account_number_entry.pack(pady=5)

        ttk.Button(self.login_frame, text="Login", command=self.login).pack(pady=20)
        
        self.error_label = ttk.Label(self.login_frame, text="", foreground="red")
        self.error_label.pack()

    def login(self):
        """Authenticates the user and switches to the account details view."""
        name = self.name_entry.get()
        account_number = self.account_number_entry.get()

        if not name or not account_number:
            self.error_label.config(text="Please enter both name and account number.")
            return

        if account_number in self.accounts:
            account = self.accounts[account_number]
            if account.account_holder.lower() == name.lower():
                self.current_account = account
                self.show_account_details()
            else:
                self.error_label.config(text="Account holder name does not match the account number.")
        else:
            self.error_label.config(text="Invalid account number.")

    def logout(self):
        """Logs the current user out and returns to the login screen."""
        self.current_account = None
        self.create_login_widgets()

    def show_account_details(self):
        """Displays the account details and action buttons for the logged-in user."""
        self.login_frame.destroy()

        self.details_frame = ttk.Frame(self.master, padding="20")
        self.details_frame.pack(fill="both", expand=True)

        ttk.Label(self.details_frame, text="Account Details", font=("Arial", 16, "bold")).pack(pady=10)
        
        ttk.Label(self.details_frame, text=f"Account Holder: {self.current_account.account_holder}").pack(anchor="w", pady=2)
        ttk.Label(self.details_frame, text=f"Account Number: {self.current_account.account_number}").pack(anchor="w", pady=2)
        
        self.balance_label = ttk.Label(self.details_frame, text=f"Balance: ${self.current_account.balance:.2f}", font=("Arial", 12, "bold"))
        self.balance_label.pack(anchor="w", pady=10)
        
        button_frame = ttk.Frame(self.details_frame)
        button_frame.pack(pady=20)

        ttk.Button(button_frame, text="Deposit", command=self.deposit_window).pack(side="left", padx=5)
        ttk.Button(button_frame, text="Withdraw", command=self.withdraw_window).pack(side="left", padx=5)
        ttk.Button(button_frame, text="Transfer", command=self.transfer_window).pack(side="left", padx=5)
        ttk.Button(self.details_frame, text="Logout", command=self.logout).pack(pady=10)
        
    def update_balance_label(self):
        """Updates the balance display on the main account details page."""
        self.balance_label.config(text=f"Balance: ${self.current_account.balance:.2f}")

    def deposit_window(self):
        """Opens a new window for making a deposit."""
        self.action_window("Deposit")

    def withdraw_window(self):
        """Opens a new window for making a withdrawal."""
        self.action_window("Withdraw")

    def transfer_window(self):
        """Opens a new window for making a transfer."""
        transfer_win = tk.Toplevel(self.master)
        transfer_win.title("Transfer Funds")
        transfer_win.geometry("350x200")

        ttk.Label(transfer_win, text="Recipient Account Number:").pack(pady=(10, 2))
        recipient_entry = ttk.Entry(transfer_win, width=30)
        recipient_entry.pack(pady=(0, 10))

        ttk.Label(transfer_win, text="Amount to Transfer:").pack(pady=(0, 2))
        amount_entry = ttk.Entry(transfer_win, width=30)
        amount_entry.pack(pady=(0, 10))

        def handle_transfer():
            recipient_number = recipient_entry.get()
            amount_str = amount_entry.get()

            if not recipient_number or not amount_str:
                messagebox.showerror("Error", "All fields are required.", parent=transfer_win)
                return
            
            if recipient_number not in self.accounts:
                messagebox.showerror("Error", "Recipient account not found.", parent=transfer_win)
                return
            
            if recipient_number == self.current_account.account_number:
                messagebox.showerror("Error", "Cannot transfer to the same account.", parent=transfer_win)
                return

            try:
                amount = float(amount_str)
            except ValueError:
                messagebox.showerror("Error", "Invalid amount. Please enter a number.", parent=transfer_win)
                return

            recipient_account = self.accounts[recipient_number]

            success = self.current_account.transfer(recipient_account, amount)
            if success:
                messagebox.showinfo("Success", f"${amount:.2f} transferred to {recipient_account.account_holder}.", parent=transfer_win)
            else:
                messagebox.showerror("Error", "Transfer failed due to insufficient funds or invalid amount.", parent=transfer_win)
            
            self.update_balance_label()
            transfer_win.destroy()

        ttk.Button(transfer_win, text="Confirm Transfer", command=handle_transfer).pack(pady=10)

    def action_window(self, action_type):
        """
        Creates a generic window for deposit and withdrawal actions.
        This consolidates repeated code.
        """
        action_win = tk.Toplevel(self.master)
        action_win.title(f"{action_type} Funds")
        action_win.geometry("300x150")
        
        ttk.Label(action_win, text=f"Enter amount to {action_type.lower()}:").pack(pady=10)
        amount_entry = ttk.Entry(action_win, width=20)
        amount_entry.pack()
        
        def handle_action():
            amount_str = amount_entry.get()
            if not amount_str:
                messagebox.showerror("Error", "Please enter an amount.", parent=action_win)
                return
            
            try:
                amount = float(amount_str)
            except ValueError:
                messagebox.showerror("Error", "Invalid amount. Please enter a number.", parent=action_win)
                return

            if action_type == "Deposit":
                success = self.current_account.deposit(amount)
                if success:
                    messagebox.showinfo("Success", f"${amount:.2f} deposited.", parent=action_win)
                else:
                    messagebox.showerror("Error", "Invalid amount. Please enter a positive number.", parent=action_win)
            
            elif action_type == "Withdraw":
                success = self.current_account.withdraw(amount)
                if success:
                    messagebox.showinfo("Success", f"${amount:.2f} withdrawn.", parent=action_win)
                else:
                    messagebox.showerror("Error", "Withdrawal failed due to insufficient funds or invalid amount.", parent=action_win)
            
            self.update_balance_label()
            action_win.destroy()

        ttk.Button(action_win, text="Confirm", command=handle_action).pack(pady=10)

# --- Main Execution Block ---
if __name__ == "__main__":
    root = tk.Tk()
    app = BankingApp(root)
    root.mainloop()
