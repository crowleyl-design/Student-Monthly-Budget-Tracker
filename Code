from dataclasses import dataclass
from drafter import *
from bakery import assert_equal

set_site_information(
    author="crowleyl@udel.edu",
    description="A monthly budget and expense tracker for college students.",
    sources=["Drafter documentation", "CISC106 lecture notes"],
    planning=["Part 0 and Part 1 Planning Documents"],
    links=["https://github.com/"]
)
set_website_title("Student Budget Tracker")

CATEGORIES = ["Food", "School", "Rent", "Fun"]
FILTER_OPTIONS = ["All", "Food", "School", "Rent", "Fun"]


@dataclass
class Expense:
    """Stores one expense with its name, category, cost, and need status."""
    name: str
    category: str
    cost: float
    is_essential: bool


@dataclass
class State:
    """Stores the user name, budget, expense list, filter, and message."""
    user_name: str
    monthly_budget: float
    expenses: list[Expense]
    selected_category: str
    status_message: str


def calculate_total_spent(expenses: list[Expense]) -> float:
    """Adds up and returns the total cost of all expenses in the list."""
    total = 0.0
    for item in expenses:
        total = total + item.cost
    return total


def get_budget_warning(total_spent: float, budget: float) -> str:
    """Checks how much money was spent and returns a warning message."""
    if budget <= 0:
        return "Please set a monthly budget greater than $0."
    elif total_spent > budget:
        over = total_spent - budget
        return "Warning: You are over budget by $" + str(over) + "!"
    elif total_spent >= budget * 0.8:
        return "Careful: You have used at least 80% of your budget."
    else:
        return "Good job! You are under your monthly budget."


def string_to_float(text: str, default_number: float) -> float:
    """Converts a string of digits to a float or returns the default."""
    if text.isdigit():
        return float(text)
    else:
        return default_number


@route
def index(state: State) -> Page:
    """Shows the home page with budget totals and buttons to other pages."""
    total_spent = calculate_total_spent(state.expenses)
    remaining = state.monthly_budget - total_spent
    warning = get_budget_warning(total_spent, state.monthly_budget)

    return Page(state, [
        Header("Student Monthly Budget Tracker"),
        "Welcome, " + state.user_name + "!",
        "Status: " + state.status_message,
        "Monthly Budget: $" + str(state.monthly_budget),
        "Total Spent: $" + str(total_spent),
        "Remaining Money: $" + str(remaining),
        "Budget Check: " + warning,
        Button("Set Budget", budget_page),
        Button("Add Expense", add_expense_page),
        Button("View Expenses", show_all_expenses),
        Button("Spending Summary", summary_page)
    ])


@route
def budget_page(state: State) -> Page:
    """Shows the page where the user can change their name and budget."""
    return Page(state, [
        Header("Set Your Monthly Budget"),
        "Your Name:",
        TextBox("new_name", state.user_name),
        "Monthly Budget Whole Dollar Amount ($):",
        TextBox("new_budget", str(int(state.monthly_budget))),
        Button("Save Budget", save_budget),
        Button("Back to Home", index)
    ])


@route
def save_budget(state: State, new_name: str, new_budget: str) -> Page:
    """Saves the new user name and budget into state and goes home."""
    if new_name != "":
        state.user_name = new_name

    state.monthly_budget = string_to_float(new_budget, state.monthly_budget)
    msg = "Budget updated to $" + str(state.monthly_budget) + "."
    state.status_message = msg
    return index(state)


@route
def add_expense_page(state: State) -> Page:
    """Shows the input boxes to add a new expense to the tracker."""
    return Page(state, [
        Header("Add a New Expense"),
        "Expense Name:",
        TextBox("item_name", ""),
        "Cost in Dollars ($):",
        TextBox("item_cost", "0"),
        "Category:",
        SelectBox("item_category", CATEGORIES, "Food"),
        "Check box if this is an essential need:",
        CheckBox("is_essential", False),
        Button("Submit Expense", submit_expense),
        Button("Back to Home", index)
    ])


@route
def submit_expense(state: State, item_name: str, item_cost: str,
                   item_category: str, is_essential: bool) -> Page:
    """Adds a new Expense object to the state list and shows the list."""
    if item_name == "":
        item_name = "Unnamed Item"

    cost_number = string_to_float(item_cost, 0.0)
    new_expense = Expense(item_name, item_category, cost_number, is_essential)
    state.expenses.append(new_expense)
    state.status_message = "Added " + item_name + " to your expenses."
    return view_expenses(state, state.selected_category)


@route
def show_all_expenses(state: State) -> Page:
    """Opens the expense list page with the current category filter."""
    return view_expenses(state, state.selected_category)


@route
def view_expenses(state: State, category_filter: str) -> Page:
    """Shows the list of expenses that match the chosen category filter."""
    state.selected_category = category_filter
    page_items = [
        Header("Expense List (" + state.selected_category + ")"),
        "Pick a category to filter:",
        SelectBox("category_filter", FILTER_OPTIONS, state.selected_category),
        Button("Apply Filter", view_expenses)
    ]

    category_total = 0.0
    count = 0

    for item in state.expenses:
        if state.selected_category == "All" or item.category == category_filter:
            if item.is_essential:
                need_text = "Need"
            else:
                need_text = "Want"

            row = item.name + " - " + item.category + " - $" + str(item.cost)
            row = row + " (" + need_text + ")"
            page_items.append(row)
            category_total = category_total + item.cost
            count = count + 1

    if count == 0:
        page_items.append("No expenses logged in this category yet.")
    else:
        page_items.append("Category Total: $" + str(category_total))

    page_items.append(Button("Clear All Expenses", clear_expenses))
    page_items.append(Button("Back to Home", index))
    return Page(state, page_items)


@route
def clear_expenses(state: State) -> Page:
    """Removes all expenses from the state list and goes back home."""
    state.expenses = []
    state.selected_category = "All"
    state.status_message = "Cleared all expenses."
    return index(state)


@route
def summary_page(state: State) -> Page:
    """Shows how much was spent on needs versus wants and the biggest item."""
    needs_total = 0.0
    wants_total = 0.0
    biggest_name = "None"
    biggest_cost = 0.0

    for item in state.expenses:
        if item.is_essential:
            needs_total = needs_total + item.cost
        else:
            wants_total = wants_total + item.cost

        if item.cost > biggest_cost:
            biggest_cost = item.cost
            biggest_name = item.name

    total_spent = needs_total + wants_total
    warning = get_budget_warning(total_spent, state.monthly_budget)
    biggest_text = biggest_name + " ($" + str(biggest_cost) + ")"

    return Page(state, [
        Header("Spending Summary"),
        "Total Spent on Needs: $" + str(needs_total),
        "Total Spent on Wants: $" + str(wants_total),
        "Overall Total Spent: $" + str(total_spent),
        "Biggest Expense: " + biggest_text,
        "Status: " + warning,
        Button("Add Another Expense", add_expense_page),
        Button("Back to Home", index)
    ])


# Unit tests for helper functions and routes
assert_equal(calculate_total_spent([]), 0.0)
assert_equal(calculate_total_spent([Expense("Lunch", "Food", 10.0, True)]), 10.0)
assert_equal(
    get_budget_warning(100.0, 500.0),
    "Good job! You are under your monthly budget."
)
assert_equal(
    get_budget_warning(450.0, 500.0),
    "Careful: You have used at least 80% of your budget."
)
assert_equal(
    get_budget_warning(550.0, 500.0),
    "Warning: You are over budget by $50.0!"
)
assert_equal(string_to_float("200", 0.0), 200.0)
assert_equal(string_to_float("abc", 50.0), 50.0)

assert_equal(
    index(State("Student", 500.0, [], "All", "Welcome!")),
    Page(State("Student", 500.0, [], "All", "Welcome!"), [
        Header("Student Monthly Budget Tracker"),
        "Welcome, Student!",
        "Status: Welcome!",
        "Monthly Budget: $500.0",
        "Total Spent: $0.0",
        "Remaining Money: $500.0",
        "Budget Check: Good job! You are under your monthly budget.",
        Button("Set Budget", budget_page),
        Button("Add Expense", add_expense_page),
        Button("View Expenses", show_all_expenses),
        Button("Spending Summary", summary_page)
    ])
)

assert_equal(
    budget_page(State("Student", 500.0, [], "All", "Welcome!")),
    Page(State("Student", 500.0, [], "All", "Welcome!"), [
        Header("Set Your Monthly Budget"),
        "Your Name:",
        TextBox("new_name", "Student"),
        "Monthly Budget Whole Dollar Amount ($):",
        TextBox("new_budget", "500"),
        Button("Save Budget", save_budget),
        Button("Back to Home", index)
    ])
)

assert_equal(
    save_budget(State("Student", 500.0, [], "All", "Welcome!"), "Sam", "600"),
    index(State("Sam", 600.0, [], "All", "Budget updated to $600.0."))
)

assert_equal(
    add_expense_page(State("Student", 500.0, [], "All", "Welcome!")),
    Page(State("Student", 500.0, [], "All", "Welcome!"), [
        Header("Add a New Expense"),
        "Expense Name:",
        TextBox("item_name", ""),
        "Cost in Dollars ($):",
        TextBox("item_cost", "0"),
        "Category:",
        SelectBox("item_category", CATEGORIES, "Food"),
        "Check box if this is an essential need:",
        CheckBox("is_essential", False),
        Button("Submit Expense", submit_expense),
        Button("Back to Home", index)
    ])
)

assert_equal(
    submit_expense(
        State("Student", 500.0, [], "All", "Welcome!"),
        "Book", "50", "School", True
    ),
    view_expenses(
        State(
            "Student", 500.0, [Expense("Book", "School", 50.0, True)],
            "All", "Added Book to your expenses."
        ),
        "All"
    )
)

assert_equal(
    show_all_expenses(
        State("Student", 500.0, [Expense("Pizza", "Food", 15.0, False)], "Food", "OK")
    ),
    Page(
        State("Student", 500.0, [Expense("Pizza", "Food", 15.0, False)], "Food", "OK"),
        [
            Header("Expense List (Food)"),
            "Pick a category to filter:",
            SelectBox("category_filter", FILTER_OPTIONS, "Food"),
            Button("Apply Filter", view_expenses),
            "Pizza - Food - $15.0 (Want)",
            "Category Total: $15.0",
            Button("Clear All Expenses", clear_expenses),
            Button("Back to Home", index)
        ]
    )
)

assert_equal(
    clear_expenses(
        State("Student", 500.0, [Expense("Pizza", "Food", 15.0, False)], "Food", "OK")
    ),
    index(State("Student", 500.0, [], "All", "Cleared all expenses."))
)

assert_equal(
    summary_page(
        State("Student", 500.0, [Expense("Rent", "Rent", 300.0, True)], "All", "OK")
    ),
    Page(
        State("Student", 500.0, [Expense("Rent", "Rent", 300.0, True)], "All", "OK"),
        [
            Header("Spending Summary"),
            "Total Spent on Needs: $300.0",
            "Total Spent on Wants: $0.0",
            "Overall Total Spent: $300.0",
            "Biggest Expense: Rent ($300.0)",
            "Status: Good job! You are under your monthly budget.",
            Button("Add Another Expense", add_expense_page),
            Button("Back to Home", index)
        ]
    )
)

start_server(State("Student", 500.0, [], "All", "Welcome!"))
