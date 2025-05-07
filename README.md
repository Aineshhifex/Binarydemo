# Binarydemo
Trading
git clone https://github.com/your-username/binary-options-demo.git
cd binary-options-demo
// server.js const express = require('express'); const sqlite3 = require('sqlite3').verbose(); const cors = require('cors'); const path = require('path'); const app = express();

app.use(cors()); app.use(express.json()); app.use(express.static('public'));

const db = new sqlite3.Database('./db.sqlite');

// Initialize DB db.serialize(() => { db.run(CREATE TABLE IF NOT EXISTS users ( id INTEGER PRIMARY KEY AUTOINCREMENT, username TEXT UNIQUE, balance REAL DEFAULT 1000 ));

db.run(CREATE TABLE IF NOT EXISTS trades ( id INTEGER PRIMARY KEY AUTOINCREMENT, user_id INTEGER, asset TEXT, direction TEXT, amount REAL, open_price REAL, close_price REAL, result TEXT, timestamp DATETIME DEFAULT CURRENT_TIMESTAMP )); });

// Simulated price (just a random walk) let currentPrice = 100; setInterval(() => { currentPrice += (Math.random() - 0.5) * 2; currentPrice = parseFloat(currentPrice.toFixed(4)); }, 1000);

// API to get current price app.get('/api/price', (req, res) => { res.json({ price: currentPrice }); });

// API to create new user app.post('/api/user', (req, res) => { const { username } = req.body; db.run(INSERT OR IGNORE INTO users (username) VALUES (?), [username], function (err) { if (err) return res.status(500).json({ error: err.message });

db.get(`SELECT * FROM users WHERE username = ?`, [username], (err, row) => {
  if (err) return res.status(500).json({ error: err.message });
  res.json(row);
});

}); });

// API to submit trade app.post('/api/trade', (req, res) => { const { user_id, direction, amount } = req.body; const openPrice = currentPrice; const expiry = 5 * 1000; // 5 seconds expiry for demo

db.get(SELECT balance FROM users WHERE id = ?, [user_id], (err, user) => { if (err || !user || user.balance < amount) { return res.status(400).json({ error: 'Invalid user or insufficient balance' }); }

db.run(`UPDATE users SET balance = balance - ? WHERE id = ?`, [amount, user_id]);

setTimeout(() => {
  const closePrice = currentPrice;
  const win = (direction === 'up' && closePrice > openPrice) ||
              (direction === 'down' && closePrice < openPrice);

  const result = win ? 'win' : 'lose';
  const payout = win ? amount * 1.8 : 0;

  db.run(`INSERT INTO trades (user_id, asset, direction, amount, open_price, close_price, result)
          VALUES (?, 'EUR/USD', ?, ?, ?, ?, ?)`,
    [user_id, direction, amount, openPrice, closePrice, result]);

  if (payout > 0) {
    db.run(`UPDATE users SET balance = balance + ? WHERE id = ?`, [payout, user_id]);
  }
}, expiry);

res.json({ message: 'Trade placed!', openPrice });

}); });

// Get user balance app.get('/api/user/:id', (req, res) => { db.get(SELECT * FROM users WHERE id = ?, [req.params.id], (err, row) => { if (err) return res.status(500).json({ error: err.message }); res.json(row); }); });

// Serve HTML frontend app.get('/', (req, res) => { res.sendFile(path.join(__dirname, 'public', 'index.html')); });

app.listen(3000, () => { console.log('Server running on http://localhost:3000'); });

