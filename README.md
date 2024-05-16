# rainfall-prediction
import tkinter as tk
from tkinter import ttk
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt

# Read the original dataset
data = pd.read_csv("Thiruvananthapuramweather.csv")
data

# Cleaning the dataset
data = data.drop(["Events", "SeaLevelPressureHighInches", "SeaLevelPressureLowInches"], axis=1)
data = data.replace("T", 0.0)
data = data.replace("-", 0.0)

# Convert 'Date' column to datetime format
data['Date'] = pd.to_datetime(data['Date'], format='%d/%m/%y')

# Set the 'Date' column as the index
data.set_index('Date', inplace=True)

X = data.drop(['PrecipitationSumInches'], axis=1)
Y = data['PrecipitationSumInches']

# Split the data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, Y, test_size=0.2, random_state=42)
# Create and fit the linear regression model
clf = LinearRegression()
clf.fit(X_train, y_train)

import matplotlib.pyplot as plt

# Predict the rainfall for the entire dataset
y_pred = clf.predict(X)

# Create a DataFrame for visualization
result_df = pd.DataFrame({'Actual': Y, 'Predicted': y_pred})

# Sort the DataFrame based on the index (date)
result_df = result_df.sort_index()

# Plotting the line graph
plt.figure(figsize=(12, 6))
plt.plot(result_df.index, result_df['Actual'], label='Actual Rainfall', marker='o')
plt.plot(result_df.index, result_df['Predicted'], label='Predicted Rainfall', marker='o')
plt.title('Actual vs Predicted Rainfall Over Time')
plt.xlabel('Date')
plt.ylabel('Rainfall (inches)')
plt.legend()
plt.show()

# Function to predict precipitation for the given day
def predict_precipitation():
    try:
        day_value = int(day_input.get())
        year_value = int(year_input.get())
        month_value = int(month_var.get())

        # Create a new date using the entered day, month, and year
        input_date = pd.Timestamp(year=year_value, month=month_value, day=day_value)

        # Check if the input date is beyond the last date in the dataset
        if input_date > X.index.max():
            result_label.config(text=f'Predicting for future dates: The selected date is beyond the last date in the dataset.')

        else:
            # Find the corresponding row for the entered date
            input_values = X.loc[[input_date]]

            if not input_values.empty:
                precipitation_prediction = clf.predict(input_values)

                result_label.config(text=f'Predicted Precipitation for {input_date.date()}: {precipitation_prediction[0]:.2f} inches')

            else:
                result_label.config(text=f'Invalid date. Please enter a valid date.')
                return None

    except ValueError as e:
        result_label.config(text=f'Error: {str(e)}')
        return None
        
# Calculate R-squared score
from sklearn.metrics import r2_score
r2 = r2_score(Y, clf.predict(X))
print(f'R-squared Score: {r2:.2f}')
        
 # Create the main window
root = tk.Tk()
root.title("Rainfall Prediction Thiruvananthapuram")

# Create input labels and entry widgets for day, month, and year
day_label = ttk.Label(root, text="Enter Day for rainfall prediction")
day_label.grid(column=0, row=0, padx=10, pady=5, sticky=tk.W)
day_input = ttk.Entry(root)
day_input.grid(column=1, row=0, padx=10, pady=5)

month_label = ttk.Label(root, text="Enter Month (MM)")
month_label.grid(column=0, row=1, padx=10, pady=5, sticky=tk.W)
month_var = tk.StringVar()
month_input = ttk.Combobox(root, textvariable=month_var, values=[str(i).zfill(2) for i in range(1, 13)])
month_input.grid(column=1, row=1, padx=10, pady=5)
month_input.set("01")

year_label = ttk.Label(root, text="Enter Year")
year_label.grid(column=0, row=2, padx=10, pady=5, sticky=tk.W)
year_input = ttk.Entry(root)
year_input.grid(column=1, row=2, padx=10, pady=5)

# Create a button to trigger the prediction
predict_button = ttk.Button(root, text="Predict Precipitation", command=predict_precipitation)
predict_button.grid(column=0, row=3, columnspan=2, pady=10)

# Create a label to display the prediction result
result_label = ttk.Label(root, text="")
result_label.grid(column=0, row=4, columnspan=2, pady=10)

# Start the main event loop
root.mainloop()       

    
