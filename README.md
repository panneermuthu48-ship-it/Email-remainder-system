# Email-remainder-system
require('dotenv').config(); const express = require('express'); const bodyParser = require('body-parser'); const nodemailer = require('nodemailer'); const cron = require('node-cron');

const app = express(); app.use(bodyParser.json());

const reminders = []; // in-memory array for reminders

// Setup email transporter const transporter = nodemailer.createTransport({ service: 'gmail', auth: { user: process.env.EMAIL_USER, pass: process.env.EMAIL_PASS, }, });

// Test route app.get('/', (req, res) => { res.send('Email Reminder System is running 🚀'); });

// Add reminder app.post('/addReminder', (req, res) => { const { email, subject, message, datetime } = req.body; if (!email || !subject || !message || !datetime) { return res.status(400).json({ error: 'All fields are required' }); }

reminders.push({ email, subject, message, datetime: new Date(datetime), sent: false, });

res.json({ message: 'Reminder added successfully!' }); });

// List all reminders app.get('/reminders', (req, res) => { res.json(reminders); });

// Cron job runs every minute cron.schedule('* * * * *', async () => { const now = new Date();

reminders.forEach(async (reminder) => { if (!reminder.sent && now >= reminder.datetime) { const mailOptions = { from: process.env.EMAIL_USER, to: reminder.email, subject: reminder.subject, text: reminder.message, };

  try {
    await transporter.sendMail(mailOptions);
    reminder.sent = true;
    console.log(`✅ Email sent to ${reminder.email}`);
  } catch (err) {
    console.error('❌ Error sending email:', err);
  }
}
}); });

const PORT = process.env.PORT || 5000; app.listen(PORT, () => console.log(Server running on port ${PORT}));
