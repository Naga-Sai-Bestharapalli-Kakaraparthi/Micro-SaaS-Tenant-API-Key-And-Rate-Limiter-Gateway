const express = require('express');
const app = express();

// In-memory tenant store (Mocking Database/Redis)
const TENANTS = {
    'key_free_123': { name: 'Free User', tier: 'free', limit: 5 },
    'key_pro_999': { name: 'Pro User', tier: 'pro', limit: 100 }
};

const usageStore = {};

// Rate Limiter Middleware
const rateLimiter = (req, res, next) => {
    const apiKey = req.headers['x-api-key'];
    
    if (!apiKey || !TENANTS[apiKey]) {
        return res.status(401).json({ error: "Unauthorized: Invalid or missing API Key" });
    }

    const tenant = TENANTS[apiKey];
    const now = Math.floor(Date.now() / 60000); // 1-minute window
    const trackingKey = `${apiKey}:${now}`;

    usageStore[trackingKey] = (usageStore[trackingKey] || 0) + 1;

    if (usageStore[trackingKey] > tenant.limit) {
        return res.status(429).json({
            error: "Too Many Requests",
            tier: tenant.tier,
            limit: tenant.limit,
            retryAfter: "1 minute"
        });
    }

    req.tenant = tenant;
    next();
};

app.use(rateLimiter);

app.get('/api/v1/data', (req, res) => {
    res.json({
        status: "success",
        message: "Protected data retrieved successfully.",
        tenant: req.tenant.name
    });
});

app.listen(4000, () => {
    console.log('API Gateway running on port 4000');
});
