const express = require("express");

const app = express();
app.use(express.json());

const PORT = process.env.PORT || 3000;
const VERIFY_TOKEN = process.env.VERIFY_TOKEN || "ceo-os-verify-token";

let lastWebhookEvent = null;

app.get("/", (req, res) => {
  res.send("CEO OS WhatsApp webhook is running.");
});

app.get("/webhook", (req, res) => {
  const mode = req.query["hub.mode"];
  const token = req.query["hub.verify_token"];
  const challenge = req.query["hub.challenge"];

  if (mode === "subscribe" && token === VERIFY_TOKEN) {
    return res.status(200).send(challenge);
  }

  return res.sendStatus(403);
});

app.post("/webhook", (req, res) => {
  lastWebhookEvent = req.body;
  console.log("Webhook event:", JSON.stringify(req.body, null, 2));
  res.sendStatus(200);
});

app.get("/last-webhook", (req, res) => {
  if (!lastWebhookEvent) {
    return res.status(200).json({ message: "No webhook received yet." });
  }

  return res.status(200).json(lastWebhookEvent);
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
