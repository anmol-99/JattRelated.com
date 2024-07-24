# E-commerce Store

An e-commerce web application built with Flask. This application allows users to register, log in, browse products, add products to their cart, and view their shopping cart.

## Features

- User registration and authentication
- Product listing and detail view
- Add products to the shopping cart
- View items in the shopping cart

## Technologies Used

- Python
- Flask
- SQLAlchemy
- Flask-WTF
- Flask-Login
- HTML/CSS

## Installation

1. **Clone the repository:**

    ```bash
    git clone https://github.com/your-username/ecommerce_store.git
    cd ecommerce_store
    ```

2. **Set up a virtual environment:**

    ```bash
    python3 -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    ```

3. **Install the dependencies:**

    ```bash
    pip install -r requirements.txt
    ```

4. **Create the database:**

    ```bash
    python
    >>> from app import db
    >>> db.create_all()
    >>> exit()
    ```

5. **Run the application:**

    ```bash
    python app.py
    ```

6. **Open your web browser** and go to `http://127.0.0.1:5000/` to see the application in action.

## Directory Structure


## Usage

- **Home Page:** Browse the list of products.
- **Product Page:** View details of a specific product.
- **Register:** Create a new user account.
- **Login:** Log in to an existing user account.
- **Cart:** View the shopping cart and manage cart items.

## Contributing

1. Fork the repository.
2. Create a new branch: `git checkout -b feature/your-feature-name`.
3. Make your changes and commit them: `git commit -m 'Add some feature'`.
4. Push to the branch: `git push origin feature/your-feature-name`.
5. Submit a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contact

Your Name - [your-email@example.com](mailto:your-email@example.com)

Project Link: [https://github.com/your-username/ecommerce_store](https://github.com/your-username/ecommerce_store)
