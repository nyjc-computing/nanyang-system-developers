
# Templating

## Overview

Templating is a programming pattern that allows for dynamic content generation by combining a template (a blueprint with placeholders) with specific data. This pattern is widely used in web development, string formatting, and code generation.

## Common Use Cases

### Web Templates (Jinja2)

Jinja2 is a templating engine for Python, commonly used with Flask for web development:

```python
# Flask route
@app.route('/hello/<name>')
def hello(name):
    return render_template('greeting.html', user_name=name)

# greeting.html template
<h1>Hello, {{ user_name }}!</h1>
{% if user_name == 'admin' %}
    <div>Welcome back, administrator!</div>
{% endif %}
```

### String Formatting (f-strings)

Python f-strings provide a concise way to embed expressions inside string literals:

```python
name = "Alice"
age = 25
print(f"My name is {name} and I am {age} years old")

# With expressions
price = 49.99
quantity = 3
print(f"Total cost: ${price * quantity:.2f}")
```

### SQL Query Templates

SQL queries often use parameterized templates for safety and reusability:

```python
# Using SQL parameters
query_template = "SELECT * FROM users WHERE age > ? AND city = ?"
cursor.execute(query_template, (18, "New York"))

# Using named parameters
query_template = "INSERT INTO users (name, email) VALUES (:name, :email)"
cursor.execute(query_template, {"name": "Bob", "email": "bob@example.com"})
```

### Code Generation Templates

Templates can generate boilerplate code:

```python
class_template = """
class {class_name}:
    def __init__(self, {params}):
        {init_body}
"""

def generate_class(name, fields):
    params = ", ".join(fields)
    init_body = "\n        ".join(f"self.{f} = {f}" for f in fields)
    return class_template.format(
        class_name=name,
        params=params,
        init_body=init_body
    )

# Usage
print(generate_class("Person", ["name", "age", "city"]))
```

### Email Templates

Templates are commonly used for generating emails:

```python
email_template = """
Dear {customer_name},

Thank you for your order #{order_id}. 
Your {product_name} will be shipped to:

{shipping_address}

Best regards,
The Store Team
"""

def send_order_confirmation(order):
    email_content = email_template.format(
        customer_name=order.customer.name,
        order_id=order.id,
        product_name=order.product.name,
        shipping_address=order.shipping_address
    )
    send_email(order.customer.email, "Order Confirmation", email_content)
```

## Best Practices

1. **Separation of Concerns**: Keep templates separate from application logic
2. **Sanitization**: Always sanitize user input to prevent injection attacks
3. **Caching**: Cache compiled templates for better performance
4. **Maintainability**: Keep templates simple and focused on presentation
5. **Documentation**: Document the expected variables and their formats

## Security Considerations

- Always escape user input to prevent XSS attacks
- Use parameterized queries for database operations
- Avoid executing dynamic code in templates
- Implement proper access controls for template rendering
