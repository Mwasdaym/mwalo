import express from "express";
import bodyParser from "body-parser";
import axios from "axios";
import dotenv from "dotenv";
import fs from "fs-extra";
import { nanoid } from "nanoid";

dotenv.config();

const app = express();
app.set("view engine", "ejs");
app.set("views", "./views");
app.use(bodyParser.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static("public"));

const ordersFile = "orders.json";
let orders = [];
if (fs.existsSync(ordersFile)) {
  orders = fs.readJsonSync(ordersFile);
}

// Home
app.get("/", (req, res) => res.render("home"));

// Services
app.get("/services", (req, res) => res.render("services"));

// Checkout
app.get("/checkout/:service", (req, res) => {
  res.render("checkout", { service: req.params.service });
});

// Pay route
app.post("/pay", async (req, res) => {
  const { phone, amount, service } = req.body;
  const orderId = nanoid();

  try {
    const response = await axios.post(
      process.env.PAYHERO_LIPWA_URL,
      {
        channel_id: process.env.PAYHERO_CHANNEL_ID,
        amount,
        msisdn: phone,
        reference: orderId,
        callback: process.env.PAYHERO_CALLBACK_URL,
      },
      {
        headers: {
          Authorization: `Bearer ${process.env.PAYHERO_API_KEY}`,
          "Content-Type": "application/json",
        },
      }
    );

    orders.push({ orderId, service, phone, amount, status: "pending" });
    fs.writeJsonSync(ordersFile, orders, { spaces: 2 });

    res.render("paid", { orderId });
  } catch (err) {
    console.error("Payment error:", err.response?.data || err.message);
    res.status(500).render("failed", {
      error: "Payment initiation failed. Please try again.",
    });
  }
});

// Callback
app.post("/payhero/callback", (req, res) => {
  console.log("Callback received:", req.body);
  const { reference, status } = req.body;

  orders = orders.map((order) =>
    order.orderId === reference ? { ...order, status } : order
  );
  fs.writeJsonSync(ordersFile, orders, { spaces: 2 });

  res.sendStatus(200);
});

// Test error page
app.get("/test-failed", (req, res) => {
  res.render("failed", { error: "This is a test error page." });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () =>
  console.log(`🚀 Server running at http://localhost:${PORT}`)
);
