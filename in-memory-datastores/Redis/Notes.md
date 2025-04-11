# 📘 Redis Full Notes — Web Dev Simplified

## 🔹 What is Redis?

- **Redis** is an **in-memory**, **NoSQL key-value** store.
- Unlike MongoDB or SQL databases, Redis:
  - Doesn’t use **tables** or **documents**.
  - Operates like a **giant JSON object** with simple **key-value pairs**.
- Runs on **RAM**, making it:
  - **Extremely fast**
  - **Volatile** (data lost if not persisted)
- Best suited for **caching** frequently accessed or computationally heavy data.

## 🔹 Use Case for Redis

Redis is used alongside a main DB (e.g., PostgreSQL or MongoDB) as a caching layer:

```
Client ---> Redis ---> DB (PostgreSQL/MongoDB)

If data in Redis: return instantly (~ms)
If not: fetch from DB (slow), then cache in Redis
```

## 🛠 Installation

### 🖥 Linux (Ubuntu):

```bash
sudo apt-get install redis
redis-server     # Starts Redis
redis-cli        # Opens Redis command-line interface
```

### 🪟 Windows:
- Use **WSL (Windows Subsystem for Linux)**
- Follow Linux installation steps inside WSL

## 🧪 Redis CLI Basics

### ✅ Set a value

```bash
SET name Kyle
```

### 📥 Get a value

```bash
GET name
```

### ❌ Delete a key

```bash
DEL name
```

### ❓ Check if key exists

```bash
EXISTS name
```

### 🔍 List all keys

```bash
KEYS *
```

### 🧹 Flush all keys

```bash
FLUSHALL
```

## ⏳ Expiry and TTL

### ⌛ Set expiry (in seconds)

```bash
EXPIRE name 10     # Key expires in 10s
TTL name           # Shows remaining time
```

### ✅ Set with expiry in one command

```bash
SETEX name 10 Kyle
```

## 🧮 Data Types

### 1️⃣ Strings (default)

```bash
SET age 25
GET age            # Returns string "25"
```

### 2️⃣ Lists (ordered)

```bash
LPUSH friends John     # Adds to start
RPUSH friends Mike     # Adds to end
LRANGE friends 0 -1    # Get all list items
LPOP friends           # Remove from start
RPOP friends           # Remove from end
```

📌 Useful for **queues/stacks** (e.g., caching latest 5 messages).

### 3️⃣ Sets (unordered, unique values)

```bash
SADD hobbies "Weight Lifting"
SMEMBERS hobbies       # Get all items
SREM hobbies "Weight Lifting"
```

📌 Good for unique collections (e.g., tags, interests).

### 4️⃣ Hashes (like flat JSON)

```bash
HSET person name Kyle
HSET person age 25
HGET person name
HGETALL person
HDEL person age
HEXISTS person age
```

📌 Perfect for storing **structured info** like user profiles.

## 🚀 Real World Usage: Express + Redis Caching

### 📦 Setup Redis in Node.js

```js
const redis = require("redis");
const redisClient = redis.createClient();
```

✅ Connect Redis Server:

```bash
redis-server
```

### 🧪 Caching Logic in Express.js

```js
app.get("/photos", async (req, res) => {
  const albumId = req.query.albumId;
  const redisKey = `photos:${albumId || "all"}`;

  redisClient.get(redisKey, async (err, data) => {
    if (err) throw err;

    if (data) {
      return res.json(JSON.parse(data)); // Cache hit
    }

    const result = await fetchPhotosFromAPI(albumId);
    redisClient.setex(redisKey, 3600, JSON.stringify(result)); // Cache for 1 hour
    res.json(result); // Cache miss
  });
});
```

### 🔁 Utility Function for DRY Caching

```js
async function getOrSetCache(key, cb) {
  return new Promise((resolve, reject) => {
    redisClient.get(key, async (err, data) => {
      if (err) return reject(err);
      if (data) return resolve(JSON.parse(data));
      const freshData = await cb();
      redisClient.setex(key, 3600, JSON.stringify(freshData));
      resolve(freshData);
    });
  });
}
```

### ✅ Use It Anywhere

```js
const photos = await getOrSetCache(`photos:${albumId}`, async () => {
  return fetchPhotosFromAPI(albumId);
});
```

## 🧪 Final Performance

- Without Redis: 300–500ms
- With Redis: 5–30ms
- 💥 10x–50x speed improvement

## ✅ Summary

| Feature              | Redis Support |
|----------------------|---------------|
| Key-Value Store      | ✅            |
| In-Memory (Fast)     | ✅            |
| Caching              | ✅            |
| Persistence          | ❌ (Volatile unless configured) |
| Expiry/TTL           | ✅            |
| Data Types (String, List, Set, Hash) | ✅ |
| Querying             | Limited to key-patterns (`KEYS *`) |
