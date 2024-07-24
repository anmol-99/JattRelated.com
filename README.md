ecommerce_store/
├── app.py
├── config.py
├── models.py
├── forms.py
├── templates/
│   ├── base.html
│   ├── index.html
│   ├── register.html
│   ├── login.html
│   ├── product.html
│   ├── cart.html
├── static/
│   └── style.css
└── requirements.txt
Flask==2.1.2
Flask-SQLAlchemy==2.5.1
Flask-WTF==1.0.1
Flask-Login==0.5.0
Werkzeug==2.0.2
import os

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or 'sqlite:///site.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
from flask import Flask, render_template, url_for, flash, redirect, request, session
from flask_sqlalchemy import SQLAlchemy
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField, BooleanField
from wtforms.validators import DataRequired, Length, Email, EqualTo, ValidationError
from flask_bcrypt import Bcrypt
from flask_login import LoginManager, UserMixin, login_user, current_user, logout_user, login_required

app = Flask(__name__)
app.config.from_object('config.Config')
db = SQLAlchemy(app)
bcrypt = Bcrypt(app)
login_manager = LoginManager(app)
login_manager.login_view = 'login'

from models import User, Product, CartItem
from forms import RegistrationForm, LoginForm

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

@app.route('/')
@app.route('/index')
def index():
    products = Product.query.all()
    return render_template('index.html', products=products)

@app.route('/register', methods=['GET', 'POST'])
def register():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = RegistrationForm()
    if form.validate_on_submit():
        hashed_password = bcrypt.generate_password_hash(form.password.data).decode('utf-8')
        user = User(username=form.username.data, email=form.email.data, password=hashed_password)
        db.session.add(user)
        db.session.commit()
        flash('Your account has been created! You are now able to log in', 'success')
        return redirect(url_for('login'))
    return render_template('register.html', title='Register', form=form)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user and bcrypt.check_password_hash(user.password, form.password.data):
            login_user(user, remember=form.remember.data)
            next_page = request.args.get('next')
            return redirect(next_page) if next_page else redirect(url_for('index'))
        else:
            flash('Login Unsuccessful. Please check email and password', 'danger')
    return render_template('login.html', title='Login', form=form)

@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('index'))

@app.route('/product/<int:product_id>')
def product(product_id):
    product = Product.query.get_or_404(product_id)
    return render_template('product.html', title=product.name, product=product)

@app.route('/cart')
@login_required
def cart():
    cart_items = CartItem.query.filter_by(user_id=current_user.id).all()
    return render_template('cart.html', cart_items=cart_items)

@app.route('/add_to_cart/<int:product_id>')
@login_required
def add_to_cart(product_id):
    product = Product.query.get_or_404(product_id)
    cart_item = CartItem(user_id=current_user.id, product_id=product.id)
    db.session.add(cart_item)
    db.session.commit()
    flash('Product added to cart', 'success')
    return redirect(url_for('cart'))

if __name__ == '__main__':
    app.run(debug=True)
from datetime import datetime
from app import db, login_manager
from flask_login import UserMixin

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

class User(db.Model, UserMixin):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(20), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    image_file = db.Column(db.String(20), nullable=False, default='default.jpg')
    password = db.Column(db.String(60), nullable=False)
    cart_items = db.relationship('CartItem', backref='buyer', lazy=True)

    def __repr__(self):
        return f"User('{self.username}', '{self.email}', '{self.image_file}')"

class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    description = db.Column(db.Text, nullable=False)
    price = db.Column(db.Float, nullable=False)
    image_file = db.Column(db.String(20), nullable=False, default='default.jpg')

    def __repr__(self):
        return f"Product('{self.name}', '{self.price}')"

class CartItem(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
    date_added = db.Column(db.DateTime, nullable=False, default=datetime.utcnow)

    def __repr__(self):
        return f"CartItem('{self.user_id}', '{self.product_id}', '{self.date_added}')"
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField, BooleanField
from wtforms.validators import DataRequired, Length, Email, EqualTo, ValidationError
from models import User

class RegistrationForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired(), Length(min=2, max=20)])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    confirm_password = PasswordField('Confirm Password', validators=[DataRequired(), EqualTo('password')])
    submit = SubmitField('Sign Up')

    def validate_username(self, username):
        user = User.query.filter_by(username=username.data).first()
        if user:
            raise ValidationError('That username is taken. Please choose a different one.')

    def validate_email(self, email):
        user = User.query.filter_by(email=email.data).first()
        if user:
            raise ValidationError('That email is already in use. Please choose a different one.')

class LoginForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember = BooleanField('Remember Me')
    submit = SubmitField('Login')
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <nav>
        <a href="{{ url_for('index') }}">Home</a>
        {% if current_user.is_authenticated %}
            <a href="{{ url_for('logout') }}">Logout</a>
            <a href="{{ url_for('cart') }}">Cart</a>
        {% else %}
            <a href="{{ url_for('login') }}">Login</a>
            <a href="{{ url_for('register') }}">Register</a>
        {% endif %}
    </nav>
    <div class="container">
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <div class="alert alert-{{ category }}">{{ message }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </div>
</body>
</html>
{% extends "base.html" %}

{% block content %}
<h1>Welcome to the E-commerce Store</h1>
<div class="products">
    {% for product in products %}
        <div class="product">
            <h2><a href="{{ url_for('product', product_id=product.id) }}">{{ product.name }}</a></h2>
            <p>{{ product.description }}</p>
            <p>${{ product.price }}</p>
        </div>
    {% endfor %}
</div>
{% endblock %}
{% extends "base.html" %}

{% block content %}
<h2>Register</h2>
<form method="POST" action="{{ url_for('register') }}">
    {{ form.hidden_tag() }}
    <div>
        {{ form.username.label }} {{ form.username }}
    </div>
    <div>
        {{ form.email.label }} {{ form.email }}
    </div>
    <div>
        {{ form.password.label }} {{ form.password }}
    </div>
    <div>
        {{ form.confirm_password.label }} {{ form.confirm_password }}
    </div>
    <div>
        {{ form.submit }}
    </div>
</form>
{% endblock %}
{% extends "base.html" %}

{% block content %}
<h2>Login</h2>
<form method="POST" action="{{ url_for('login') }}">
    {{ form.hidden_tag() }}
    <div>
        {{ form.email.label }} {{ form.email }}
    </div>
    <div>
        {{ form.password.label }} {{ form.password }}
    </div>
    <div>
        {{ form.remember }} {{ form.remember.label }}
    </div>
    <div>
        {{ form.submit }}
    </div>
</form>
{% endblock %}
{% extends "base.html" %}

{% block content %}
<h2>Login</h2>
<form method="POST" action="{{ url_for('login') }}">
    {{ form.hidden_tag() }}
    <div>
        {{ form.email.label }} {{ form.email }}
    </div>
    <div>
        {{ form.password.label }} {{ form.password }}
    </div>
    <div>
        {{ form.remember }} {{ form.remember.label }}
    </div>
    <div>
        {{ form.submit }}
    </div>
</form>
{% endblock %}
{% extends "base.html" %}

{% block content %}
<h2>Your Shopping Cart</h2>
<ul>
    {% for item in cart_items %}
        <li>{{ item.product.name }} - ${{ item.product.price }}</li>
    {% endfor %}
</ul>
{% endblock %}
body {
    font-family: Arial, sans-serif;
}

.container {
    width: 80%;
    margin: 0 auto;
    padding: 20px;
}

nav {
    text-align: center;
    margin-bottom: 20px;
}

nav a {
    margin: 0 10px;
    text-decoration: none;
    color: #000;
}

.products {
    display: flex;
    flex-wrap: wrap;
}

.product {
    flex: 1;
    margin: 10px;
    padding: 10px;
    border: 1px solid #ddd;
}

.alert {
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid transparent;
    border-radius: 4px;
}

.alert-success {
    color: #3c763d;
    background-color: #dff0d8;
    border-color: #d6e9c6;
}

.alert-danger {
    color: #a94442;
    background-color: #f2dede;
    border-color: #ebccd1;
}
from app import db
db.create_all()
python3 -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
pip install -r requirements.txt
python
>>> from app import db
>>> db.create_all()
>>> exit()
python app.py
