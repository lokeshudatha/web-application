# web-application
from flask import Flask, render_template, request, redirect, url_for, flash
import sqlite3
import re

app = Flask(__name__)
app.secret_key = 'your_secret_key'

# Database initialization
conn = sqlite3.connect('userdata.db', check_same_thread=False)
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS users (
             id INTEGER PRIMARY KEY,
             name TEXT NOT NULL,
             email TEXT NOT NULL,
             age INTEGER NOT NULL,
             dob DATE NOT NULL)''')
conn.commit()


# Validate email format
def validate_email(email):
    return re.match(r"[^@]+@[^@]+\.[^@]+", email)


# Validate age as a positive integer
def validate_age(age):
    try:
        return int(age) > 0
    except ValueError:
        return False


# Home page with user input form
@app.route('/')
def home():
    return render_template('index.html')


# Add user data to the database
@app.route('/add_user', methods=['POST'])
def add_user():
    name = request.form['name']
    email = request.form['email']
    age = request.form['age']
    dob = request.form['dob']

    if not (name and email and age and dob):
        flash('All fields are required', 'error')
        return redirect(url_for('home'))

    if not validate_email(email):
        flash('Invalid email format', 'error')
        return redirect(url_for('home'))

    if not validate_age(age):
        flash('Age must be a positive integer', 'error')
        return redirect(url_for('home'))

    conn.execute("INSERT INTO users (name, email, age, dob) VALUES (?, ?, ?, ?)",
                 (name, email, age, dob))
    conn.commit()
    flash('User added successfully', 'success')
    return redirect(url_for('home'))


# Display user data in tabular format
@app.route('/users')
def users():
    cursor = conn.execute("SELECT * FROM users")
    users = cursor.fetchall()
    return render_template('users.html', users=users)


if __name__ == '__main__':
    app.run(debug=True)


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>User Input Form</title>
</head>
<body>
    <h1>User Input Form</h1>
    {% with messages = get_flashed_messages() %}
        {% if messages %}
            <ul>
                {% for message in messages %}
                    <li>{{ message }}</li>
                {% endfor %}
            </ul>
        {% endif %}
    {% endwith %}
    <form action="/add_user" method="post">
        <label for="name">Name:</label>
        <input type="text" id="name" name="name" required><br><br>
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required><br><br>
        <label for="age">Age:</label>
        <input type="number" id="age" name="age" required><br><br>
        <label for="dob">Date of Birth:</label>
        <input type="date" id="dob" name="dob" required><br><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>User Data</title>
</head>
<body>
    <h1>User Data</h1>
    <table border="1">
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Age</th>
            <th>Date of Birth</th>
        </tr>
        {% for user in users %}
            <tr>
                <td>{{ user[0] }}</td>
                <td>{{ user[1] }}</td>
                <td>{{ user[2] }}</td>
                <td>{{ user[3] }}</td>
                <td>{{ user[4] }}</td>
            </tr>
        {% endfor %}
    </table>
</body>
</html>
