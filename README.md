# autoserach_customers asscoited with client name


{% extends 'base.html' %}
{% load static %}

{% block main %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Search Customers</title>

    <!-- Include jQuery -->
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
</head>
<body>
    <h1>Search Customers</h1>

    <!-- Search Form -->
    <form method="get" action="{% url 'search_customers' %}">
        <label for="customer_name">Customer Name:</label>
        <input type="text" id="customer_name" name="customer_name" placeholder="Enter customer name" value="{{ request.GET.customer_name }}">

        <label for="customer_address">Address:</label>
        <input type="text" id="customer_address" name="customer_address" placeholder="Enter customer address" value="{{ request.GET.customer_address }}">

        <label for="customer_contact">Contact:</label>
        <input type="text" id="customer_contact" name="customer_contact" placeholder="Enter customer contact" value="{{ request.GET.customer_contact }}">

        <label for="customer_email">Email:</label>
        <input type="text" id="customer_email" name="customer_email" placeholder="Enter customer email" value="{{ request.GET.customer_email }}">

        <label for="client_name">Client Name:</label>
        <input type="text" id="client_name" name="client_name" placeholder="Enter client name" value="{{ request.GET.client_name }}">

        <button type="submit">Search</button>
    </form>

    <!-- Placeholder for displaying search results dynamically -->
    <div id="search-results"></div>

    <!-- Order Item Table (for adding items to the order) -->
    <div class="form-group">
        <label for="items">Order Items:</label>
        <table id="order-items-table">
            <thead>
                <tr>
                    <th>Item</th>
                    <th>Unit Price</th>
                    <th>Quantity</th>
                    <th>Total Price</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>
                        <select name="item[]" class="form-control">
                            <option value="Fried Rice">Fried Rice</option>
                            <option value="Coke">Coke</option>
                            <option value="Biryani">Biryani</option>
                            <option value="Paneer Lababdar">Paneer Lababdar</option>
                        </select>
                    </td>
                    <td><input type="number" name="unit_price[]" class="form-control" required step="0.01"></td>
                    <td><input type="number" name="quantity[]" class="form-control" required min="1"></td>
                    <td><input type="text" name="total_price[]" class="form-control" readonly></td>
                </tr>
            </tbody>
        </table>
        <button type="button" id="add-item-btn" class="btn btn-add">Add Item</button>
    </div>
     <!-- Order Form Details -->
     <div class="form-group">
        <label for="order_status">Order Status:</label>
        <select name="order_status" class="form-control" required>
            <option value="Completed">Completed</option>
            <option value="Pending">Pending</option>
            <option value="Other">Entered</option>
        </select>
    </div>
    <div class="form-group">
        <label for="shipped_by">Shipped By:</label>
        <select name="shipped_by" class="form-control" required>
            <option value="FedEx">FedEx</option>
            <option value="DHL">DHL</option>
            <option value="UPS">UPS</option>
            <option value="USPS">USPS</option>
            <option value="Other">Other</option>
        </select>
    </div>
    <div class="form-group">
        <label for="billing_status">Billing Status:</label>
        <select name="billing_status" class="form-control" required>
            <option value="Paid">Paid</option>
            <option value="Pending">Pending</option>
            <option value="Overdue">Overdue</option>
        </select>
    </div>
    <div class="form-group">
        <label for="delivery_address">Delivery Address:</label>
        <input type="text" name="delivery_address" class="form-control" required>
    </div>
    <div class="form-group">
        <label for="payment_mode">Order Payment Mode:</label>
        <select name="payment_mode" class="form-control" required>
            <option value="Credit Card">Credit Card</option>
            <option value="Debit Card">Debit Card</option>
            <option value="PayPal">PayPal</option>
            <option value="Bank Transfer">Bank Transfer</option>
            <option value="Cash on Delivery">Cash on Delivery</option>
        </select>
    </div>
    <!-- Total Amount Section -->
    <div class="form-group">
        <label for="sub_total">Sub Total ($):</label>
        <input type="text" id="sub_total" name="sub_total" class="form-control" readonly>
    </div>
    <div class="form-group">
        <label for="taxes">Taxes ($):</label>
        <input type="text" id="taxes" name="taxes" class="form-control">
    </div>
    <div class="form-group">
        <label for="total">Total Amount ($):</label>
        <input type="text" id="total" name="total" class="form-control" required>
    </div>

    <div id="success-message" style="display:none; color: rgb(252, 165, 131); margin-bottom: 10px;">
        Order placed successfully!
    </div>

    <!-- Form Buttons -->
    <div class="form-buttons">
        <form method="POST" action="{% url 'save_order' %}" id="order-form">
            {% csrf_token %}
            <button type="submit" class="btn-prim">Save</button>
        </form>

        <button type="reset" class="btn-second">Reset</button>
        <button type="button" class="btn-delete" onclick="window.location.href='{% url 'search_customers' %}';">Cancel</button>
    </div>

    <!-- JavaScript -->
    <script type="text/javascript">
        $(document).ready(function() {
            // Trigger when user types in any search field
            $('#customer_name, #customer_address, #customer_contact, #customer_email').on('input', function() {
                var customerName = $('#customer_name').val();
                var customerAddress = $('#customer_address').val();
                var customerContact = $('#customer_contact').val();
                var customerEmail = $('#customer_email').val();
                var clientName = $('#client_name').val();  // Include client_name field value as well

                // Send an AJAX request to fetch matching customers
                $.ajax({
                    url: '{% url "search_customers" %}',  // URL to the search view
                    data: {
                        'customer_name': customerName,
                        'customer_address': customerAddress,
                        'customer_contact': customerContact,
                        'customer_email': customerEmail,
                        'client_name': clientName
                    },
                    success: function(response) {
                        // Update the result section with the returned data
                        if (response.customers.length > 0) {
                            var resultHtml = '<h2>Search Results</h2><table border="1"><thead><tr><th>Customer Name</th><th>Email</th><th>Contact</th><th>Address</th><th>Payment Method</th><th>Order Numbers</th><th>Client Name</th></tr></thead><tbody>';
                            $.each(response.customers, function(index, customer) {
                                resultHtml += '<tr class="customer-row" data-customer-id="' + customer.customer_id + '">' +
                                    '<td>' + customer.customer_name + '</td>' +
                                    '<td>' + customer.customer_email + '</td>' +
                                    '<td>' + customer.customer_contact + '</td>' +
                                    '<td>' + customer.customer_address1 + ' ' + customer.customer_address2 + ' ' + customer.customer_address3 + ' ' + customer.customer_address4 + ' ' + customer.customer_address5 + '</td>' +
                                    '<td>' + customer.customer_payment_method + '</td>' +
                                    '<td>' + (customer.order_numbers.length > 0 ? customer.order_numbers.join(', ') : 'No Orders') + '</td>' +
                                    '<td>' + customer.client_name + '</td>' +
                                    '</tr>';
                            });
                            resultHtml += '</tbody></table>';
                            $('#search-results').html(resultHtml);  // Update the results section
                        } else {
                            $('#search-results').html('<p>No customers found.</p>');  // No results found
                        }
                    },
                    error: function() {
                        $('#search-results').html('<p>There was an error while fetching the data.</p>');
                    }
                });
            });

            // When a customer row is clicked, autofill the form
            $('#search-results').on('click', '.customer-row', function() {
                var customerId = $(this).data('customer-id');
                var customerName = $(this).find('td').eq(0).text();
                var customerEmail = $(this).find('td').eq(1).text();
                var customerContact = $(this).find('td').eq(2).text();
                var customerAddress = $(this).find('td').eq(3).text();
                var clientName = $(this).find('td').eq(6).text(); // Assuming this is client_name

                // Autofill form
                $('#customer_name').val(customerName);
                $('#customer_email').val(customerEmail);
                $('#customer_contact').val(customerContact);
                $('#customer_address').val(customerAddress);
                $('#client_name').val(clientName);
            });
        });
    </script>
</body>
</html>
{% endblock %}

*********model.py ******

class Customer(models.Model):
    # Primary Key
    customer_id = models.CharField(max_length=50, primary_key=True, unique=True)

    # Foreign Keys
    client = models.ForeignKey('Client', on_delete=models.CASCADE, null=False, related_name='customers')
    created_by = models.ForeignKey('User', on_delete=models.CASCADE, null=False, related_name='created_customers')
    updated_by = models.ForeignKey('User', on_delete=models.SET_NULL, null=True, blank=True, related_name='updated_customers')
    deleted_by = models.ForeignKey('User', on_delete=models.SET_NULL, null=True, blank=True, related_name='deleted_customers')

    # Customer Details
    customer_name = models.CharField(max_length=250, null=False)
    customer_contact = models.CharField(max_length=10, null=False)
    customer_email = models.EmailField(max_length=50, null=False)
    customer_password = models.CharField(max_length=250, null=False)
    customer_address1 = models.CharField(max_length=500, null=False)
    customer_address2 = models.CharField(max_length=500, null=False)
    customer_address3 = models.CharField(max_length=500, null=False)
    customer_address4 = models.CharField(max_length=500, null=False)
    customer_address5 = models.CharField(max_length=500, null=False)
    customer_payment_method = models.CharField(max_length=100, null=False)  # Replacing dropdown with CharField

    # Audit Fields
    created_on = models.DateField(auto_now_add=True)
    updated_on = models.DateField(auto_now=True, null=True, blank=True)
    deleted_on = models.DateField(null=True, blank=True)

    def __str__(self):
        return self.customer_name


  ***************order module

  class Order(models.Model):
    # Primary Key
    order_number = models.CharField(max_length=50, primary_key=True, unique=True)

    # Foreign Keys
    client = models.ForeignKey('Client', on_delete=models.CASCADE, null=False, related_name='orders')
    customer_id = models.ForeignKey('Customer', on_delete=models.CASCADE, null=False, related_name='orders')
    created_by = models.ForeignKey('User', on_delete=models.CASCADE, null=False, related_name='created_orders')
    updated_by = models.ForeignKey('User', on_delete=models.SET_NULL, null=True, blank=True, related_name='updated_orders')
    deleted_by = models.ForeignKey('User', on_delete=models.SET_NULL, null=True, blank=True, related_name='deleted_orders')

    # Order Details
    line_number = models.CharField(max_length=50, null=True, blank=True)
    order_price = models.FloatField(null=False)
    order_payment_mode = models.CharField(max_length=100, null=False)  # Replacing dropdown with CharField
    order_payment_method = models.CharField(max_length=100, null=False)  # Replacing dropdown with CharField
    delivery_address = models.CharField(max_length=500, null=False)
    order_status = models.CharField(max_length=100, null=False)  # Replacing dropdown with CharField
    billing_status = models.CharField(max_length=100, null=False)  # Replacing dropdown with CharField
    payment_status = models.CharField(max_length=100, null=False)  # Replacing dropdown with CharField
    third_party_tracking_number = models.CharField(max_length=50, null=True, blank=True)
    shipped_by = models.CharField(max_length=200, null=True, blank=True)
    line_status = models.CharField(max_length=100, null=False)  # Replacing dropdown with CharField

    # Audit Fields
    created_on = models.DateField(auto_now_add=True)
    updated_on = models.DateField(auto_now=True, null=True, blank=True)
    deleted_on = models.DateField(null=True, blank=True)

    def __str__(self):
        return self.order_number
