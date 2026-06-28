// Vercel Serverless Function: /api/jisr-proxy
// يمرّر الطلبات إلى Jisr Open API مع جميع الهيدرات (api-key, secret, Access-Token, ...).
// مسار Jisr يُحدَّد عبر معامل ?p=/path  (مثلاً ?p=/employees أو ?p=/auth)

const JISR_BASE_URL = 'https://apis.jisr.net/api/openapi/v1';

const SKIP_HEADERS = new Set([
  'host','connection','content-length','accept-encoding','accept-language',
  'cookie','referer','origin','user-agent','sec-fetch-mode','sec-fetch-site',
  'sec-fetch-dest','sec-ch-ua','sec-ch-ua-mobile','sec-ch-ua-platform',
  'cdn-loop','forwarded','x-forwarded-for','x-forwarded-host','x-forwarded-proto',
  'x-real-ip','x-vercel-id','x-vercel-deployment-url','x-vercel-forwarded-for',
  'x-vercel-ip-city','x-vercel-ip-country','x-vercel-ip-country-region',
  'x-vercel-ip-latitude','x-vercel-ip-longitude','x-vercel-ip-timezone',
  'x-vercel-proxied-for','x-vercel-proxy-signature','x-vercel-proxy-signature-ts'
]);

module.exports = async (req, res) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Headers', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE,OPTIONS');
  res.setHeader('Access-Control-Max-Age', '86400');
  if (req.method === 'OPTIONS') { res.status(204).end(); return; }

  try {
    const qs = req.query || {};
    const apiPath = qs.p || '/employees';

    const forwardParams = Object.entries(qs)
      .filter(([k]) => k !== 'p')
      .map(([k, v]) => `${encodeURIComponent(k)}=${encodeURIComponent(v)}`)
      .join('&');

    const url = JISR_BASE_URL + apiPath + (forwardParams ? '?' + forwardParams : '');

    const fwdHeaders = {};
    for (const [k, v] of Object.entries(req.headers || {})) {
      if (!SKIP_HEADERS.has(k.toLowerCase())) fwdHeaders[k] = v;
    }
    if (!fwdHeaders['accept'])       fwdHeaders['accept']       = 'application/json';
    if (!fwdHeaders['content-type']) fwdHeaders['content-type'] = 'application/json';

    let body;
    if (req.method !== 'GET' && req.method !== 'HEAD') {
      if (req.body) body = typeof req.body === 'string' ? req.body : JSON.stringify(req.body);
    }

    const r = await fetch(url, { method: req.method, headers: fwdHeaders, body });
    const text = await r.text();

    res.setHeader('Content-Type', r.headers.get('content-type') || 'application/json');
    res.status(r.status).send(text);
  } catch (e) {
    res.status(500).json({ error: 'proxy_error', message: e.message });
  }
};
