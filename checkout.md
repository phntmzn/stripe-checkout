You can build a complete Stripe payment website using **HTML, CSS, vanilla JavaScript, and Stripe Checkout**, hosted on Netlify.

You don't need PHP or React. Use a small Netlify serverless function in JavaScript to securely communicate with Stripe.

## 1. Project structure

```text
stripe-checkout/
│
├── public/
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   ├── success.html
│   └── cancel.html
│
├── netlify/
│   └── functions/
│       └── checkout.js
│
├── package.json
└── netlify.toml
```

## 2. Install dependencies

```bash
npm init -y
npm install stripe
```

## 3. HTML — index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Stripe Checkout</title>

    <link rel="stylesheet" href="style.css">
</head>
<body>

    <main class="container">

        <h1>My Store</h1>

        <div class="product">
            <h2>Premium Access</h2>

            <p>Unlock premium features.</p>

            <h3>$9.99 USD</h3>

            <button id="checkout">
                Pay with Stripe
            </button>
        </div>

    </main>

    <script src="script.js"></script>

</body>
</html>
```

## 4. CSS — style.css

```css
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #111;
    color: white;
}

.container {
    max-width: 500px;
    margin: 100px auto;
    padding: 20px;
    text-align: center;
}

.product {
    background: #202020;
    padding: 40px;
    border-radius: 12px;
}

button {
    background: #635bff;
    color: white;
    border: none;
    padding: 15px 30px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 16px;
}

button:hover {
    background: #5145e5;
}
```

## 5. JavaScript — script.js

```javascript
// Connect checkout button to Netlify Function

const button = document.getElementById("checkout");

button.addEventListener("click", async () => {

    button.disabled = true;
    button.textContent = "Redirecting...";

    try {

        const response = await fetch(
            "/.netlify/functions/checkout",
            {
                method: "POST"
            }
        );

        const data = await response.json();

        if (!response.ok) {
            throw new Error(data.error || "Checkout failed");
        }

        // Redirect to Stripe Checkout
        window.location.href = data.url;

    } catch (error) {

        alert(error.message);

        button.disabled = false;
        button.textContent = "Pay with Stripe";

    }

});
```

## 6. Stripe backend — checkout.js

```javascript
// netlify/functions/checkout.js

const Stripe = require("stripe");

exports.handler = async (event) => {

    if (event.httpMethod !== "POST") {
        return {
            statusCode: 405,
            body: JSON.stringify({
                error: "Method not allowed"
            })
        };
    }

    try {

        const stripe = Stripe(
            process.env.STRIPE_SECRET_KEY
        );

        const session = await stripe.checkout.sessions.create({

            payment_method_types: ["card"],

            mode: "payment",

            line_items: [
                {
                    price_data: {
                        currency: "usd",

                        product_data: {
                            name: "Premium Access"
                        },

                        unit_amount: 999
                    },

                    quantity: 1
                }
            ],

            success_url:
                `${process.env.URL}/success.html`,

            cancel_url:
                `${process.env.URL}/cancel.html`

        });

        return {
            statusCode: 200,

            body: JSON.stringify({
                url: session.url
            })
        };

    } catch (error) {

        return {
            statusCode: 500,

            body: JSON.stringify({
                error: "Unable to create checkout session"
            })
        };

    }

};
```

## 7. Netlify configuration

`netlify.toml`

```toml
[build]
    publish = "public"
    functions = "netlify/functions"
```

## 8. Success and cancel pages

`success.html`

```html
<h1>Payment Successful!</h1>
<p>Thank you for your purchase.</p>
<a href="/">Return Home</a>
```

`cancel.html`

```html
<h1>Payment Cancelled</h1>
<p>Your payment was not completed.</p>
<a href="/">Try Again</a>
```

## 9. Configure Stripe

Create an account at [oai_citation:0‡stripe.com](https://stripe.com?utm_source=chatgpt.com).

In your Stripe Dashboard:

1. Open Developers → API keys.
2. Copy your test secret key.
3. Open your Netlify site settings.
4. Add an environment variable:

```text
STRIPE_SECRET_KEY=sk_test_your_key
```

5. Redeploy your website.

**Important:** Never put your Stripe secret key in HTML or frontend JavaScript. Keep it inside Netlify Functions. [oai_citation:1‡Netlify](https://www.netlify.com/knowledge-base/how-to-add-a-backend-api-endpoint-with-netlify-functions/?utm_source=chatgpt.com)

---

### What this setup supports

- HTML/CSS storefront
- Vanilla JavaScript
- Stripe-hosted checkout
- Credit and debit card payments
- One-time purchases
- Netlify deployment

You can later add subscriptions, customer accounts, webhooks, and payment history.

**Next step:** I can expand this into a complete Stripe SaaS payment system with API keys, customer dashboards, prepaid credits, and usage-based billing.