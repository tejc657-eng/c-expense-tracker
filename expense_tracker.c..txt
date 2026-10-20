
#include <stdio.h>

#define MAX_EXPENSES 100
#define EXPENSE_FILE "expenses_v2.txt"
#define BUDGET_FILE "budget.txt"

void saveExpenses(float expenses[], int categories[],
                  char dates[][11], int count) {
    FILE *fp = fopen(EXPENSE_FILE, "w");
    int i;

    if (fp == NULL) {
        printf("Error saving expenses!\n");
        return;
    }

    for (i = 0; i < count; i++) {
        fprintf(fp, "%.2f %d %s\n",
                expenses[i], categories[i], dates[i]);
    }

    fclose(fp);
}

void loadExpenses(float expenses[], int categories[],
                  char dates[][11], int *count) {
    FILE *fp = fopen(EXPENSE_FILE, "r");

    if (fp == NULL) {
        return;
    }

    while (*count < MAX_EXPENSES &&
           fscanf(fp, "%f %d %10s",
                  &expenses[*count],
                  &categories[*count],
                  dates[*count]) == 3) {

        if (expenses[*count] > 0 &&
            categories[*count] >= 1 &&
            categories[*count] <= 4) {
            (*count)++;
        }
    }

    fclose(fp);
}

void calculateTotals(float expenses[], int categories[],
                     int count, float *total,
                     float categoryTotal[]) {
    int i;

    *total = 0;

    for (i = 0; i < 5; i++) {
        categoryTotal[i] = 0;
    }

    for (i = 0; i < count; i++) {
        *total += expenses[i];
        categoryTotal[categories[i]] += expenses[i];
    }
}

void showCategory(int category) {
    if (category == 1)
        printf("Food");
    else if (category == 2)
        printf("Travel");
    else if (category == 3)
        printf("Shopping");
    else
        printf("Other");
}

void checkBudget(float total, float budget) {
    if (budget <= 0) {
        printf("Set a budget to track your spending.\n");
    } else if (total >= budget) {
        printf("WARNING: Your budget limit has been reached or exceeded!\n");
        printf("Amount over budget: Rs. %.2f\n", total - budget);
    } else {
        printf("Budget remaining: Rs. %.2f\n", budget - total);
    }
}

int main(void) {
    float expenses[MAX_EXPENSES];
    int categories[MAX_EXPENSES];
    char dates[MAX_EXPENSES][11];

    float total = 0;
    float categoryTotal[5] = {0};
    float amount, budget = 0;

    int count = 0;
    int choice, category, searchCategory;
    int i, deleteNumber, found;

    FILE *fp;

    loadExpenses(expenses, categories, dates, &count);

    fp = fopen(BUDGET_FILE, "r");
    if (fp != NULL) {
        fscanf(fp, "%f", &budget);
        fclose(fp);
    }

    calculateTotals(expenses, categories, count,
                    &total, categoryTotal);

    printf("Welcome to Expense Tracker!\n");
    printf("Previous expenses loaded: %d\n", count);

    while (1) {
        printf("\n===== EXPENSE TRACKER =====\n");
        printf("1. Add Expense\n");
        printf("2. View All Expenses\n");
        printf("3. View Category Totals\n");
        printf("4. Search by Category\n");
        printf("5. Set Monthly Budget\n");
        printf("6. Delete an Expense\n");
        printf("7. Save and Exit\n");
        printf("Enter your choice: ");

        if (scanf("%d", &choice) != 1) {
            printf("Invalid input. Please restart the program.\n");
            break;
        }

        if (choice == 1) {
            if (count >= MAX_EXPENSES) {
                printf("Expense limit reached!\n");
                continue;
            }

            printf("\nCategories:\n");
            printf("1. Food\n2. Travel\n3. Shopping\n4. Other\n");
            printf("Enter category number: ");

            if (scanf("%d", &category) != 1) {
                printf("Invalid input.\n");
                break;
            }

            if (category < 1 || category > 4) {
                printf("Invalid category!\n");
                continue;
            }

            printf("Enter expense amount: Rs. ");

            if (scanf("%f", &amount) != 1) {
                printf("Invalid input.\n");
                break;
            }

            if (amount <= 0) {
                printf("Amount must be greater than zero.\n");
                continue;
            }

            printf("Enter date (DD-MM-YYYY): ");

            if (scanf("%10s", dates[count]) != 1) {
                printf("Invalid date input.\n");
                break;
            }

            expenses[count] = amount;
            categories[count] = category;
            count++;

            calculateTotals(expenses, categories, count,
                            &total, categoryTotal);

            saveExpenses(expenses, categories, dates, count);

            printf("Expense added successfully!\n");
            printf("Total expenses: Rs. %.2f\n", total);
            checkBudget(total, budget);
        }

        else if (choice == 2) {
            if (count == 0) {
                printf("No expenses recorded yet.\n");
            } else {
                printf("\n===== ALL EXPENSES =====\n");

                for (i = 0; i < count; i++) {
                    printf("%d. Rs. %.2f | ",
                           i + 1, expenses[i]);
                    showCategory(categories[i]);
                    printf(" | Date: %s\n", dates[i]);
                }

                printf("------------------------\n");
                printf("Total: Rs. %.2f\n", total);
                checkBudget(total, budget);
            }
        }

        else if (choice == 3) {
            printf("\n===== CATEGORY TOTALS =====\n");
            printf("Food: Rs. %.2f\n", categoryTotal[1]);
            printf("Travel: Rs. %.2f\n", categoryTotal[2]);
            printf("Shopping: Rs. %.2f\n", categoryTotal[3]);
            printf("Other: Rs. %.2f\n", categoryTotal[4]);
            printf("Grand Total: Rs. %.2f\n", total);
        }

        else if (choice == 4) {
            printf("\nSearch category:\n");
            printf("1. Food\n2. Travel\n3. Shopping\n4. Other\n");
            printf("Enter category number: ");

            if (scanf("%d", &searchCategory) != 1) {
                printf("Invalid input.\n");
                break;
            }

            if (searchCategory < 1 || searchCategory > 4) {
                printf("Invalid category!\n");
                continue;
            }

            found = 0;

            for (i = 0; i < count; i++) {
                if (categories[i] == searchCategory) {
                    printf("Rs. %.2f | Date: %s\n",
                           expenses[i], dates[i]);
                    found = 1;
                }
            }

            if (!found) {
                printf("No expenses found in this category.\n");
            }

            printf("Category total: Rs. %.2f\n",
                   categoryTotal[searchCategory]);
        }

        else if (choice == 5) {
            printf("Enter your monthly budget: Rs. ");

            if (scanf("%f", &budget) != 1) {
                printf("Invalid input.\n");
                break;
            }

            if (budget <= 0) {
                budget = 0;
                printf("Budget must be greater than zero.\n");
                continue;
            }

            fp = fopen(BUDGET_FILE, "w");

            if (fp != NULL) {
                fprintf(fp, "%.2f\n", budget);
                fclose(fp);
                printf("Budget saved successfully!\n");
            } else {
                printf("Could not save budget to file.\n");
            }

            checkBudget(total, budget);
        }

        else if (choice == 6) {
            if (count == 0) {
                printf("There are no expenses to delete.\n");
                continue;
            }

            printf("\n===== ALL EXPENSES =====\n");

            for (i = 0; i < count; i++) {
                printf("%d. Rs. %.2f | ",
                       i + 1, expenses[i]);
                showCategory(categories[i]);
                printf(" | Date: %s\n", dates[i]);
            }

            printf("Enter the expense number to delete: ");

            if (scanf("%d", &deleteNumber) != 1) {
                printf("Invalid input.\n");
                break;
            }

            if (deleteNumber < 1 || deleteNumber > count) {
                printf("Invalid expense number!\n");
                continue;
            }

            for (i = deleteNumber - 1; i < count - 1; i++) {
                expenses[i] = expenses[i + 1];
                categories[i] = categories[i + 1];

                snprintf(dates[i], sizeof(dates[i]),
                         "%s", dates[i + 1]);
            }

            count--;

            calculateTotals(expenses, categories, count,
                            &total, categoryTotal);

            saveExpenses(expenses, categories, dates, count);

            printf("Expense deleted successfully!\n");
            printf("Updated total: Rs. %.2f\n", total);
            checkBudget(total, budget);
        }

        else if (choice == 7) {
            saveExpenses(expenses, categories, dates, count);
            printf("Expenses saved. Goodbye!\n");
            break;
        }

        else {
            printf("Invalid choice! Enter 1 to 7.\n");
        }
    }

    return 0;
}
