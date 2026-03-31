// netlify/functions/send-beoordeling.mjs
// Receives form data, generates PDFs, sends email via Resend.
//
// Environment variable required (set in Netlify dashboard):
//   RESEND_API_KEY  — your Resend API key (get free at resend.com)
//
// Optional — defaults to admin@defietsboot.nl if not set:
//   MAIL_TO         — recipient email
//   MAIL_FROM       — sender email (must be verified in Resend)

import { buildBeoordelingPdf, buildVerklaringPdf } from './pdf-builder.mjs';

const MAIL_TO   = process.env.MAIL_TO   || 'admin@defietsboot.nl';
const MAIL_FROM = process.env.MAIL_FROM || 'vaartraining@defietsboot.nl';

export default async function handler(req) {

  // ── CORS preflight ────────────────────────────────────────────────────
  if (req.method === 'OPTIONS') {
    return new Response(null, {
      status: 204,
      headers: corsHeaders(),
    });
  }

  if (req.method !== 'POST') {
    return json({ ok: false, error: 'POST only' }, 405);
  }

  // ── Parse body ────────────────────────────────────────────────────────
  let data;
  try {
    data = await req.json();
  } catch {
    return json({ ok: false, error: 'Ongeldige JSON' }, 400);
  }

  const {
    datum_training, naam_opleider, email_opleider,
    naam_trainee, email_trainee, boot, functie,
    scores = [], toelichting = '', conclusie = '',
    toelichting_aanbeveling = '',
  } = data;

  if (!naam_trainee || !naam_opleider || !functie || !conclusie) {
    return json({ ok: false, error: 'Verplichte velden ontbreken' }, 400);
  }

  const isGekwalificeerd = conclusie.toLowerCase().includes('gekwalificeerd');
  const datumNl = datum_training
    ? datum_training.split('-').reverse().join('-')
    : new Date().toLocaleDateString('nl-NL');

  // ── Generate PDFs ─────────────────────────────────────────────────────
  let beoordelingBytes, verklaringBytes;
  try {
    beoordelingBytes = await buildBeoordelingPdf(data);
    if (isGekwalificeerd) {
      verklaringBytes = await buildVerklaringPdf(data);
    }
  } catch (err) {
    console.error('PDF error:', err);
    return json({ ok: false, error: `PDF fout: ${err.message}` }, 500);
  }

  // ── Build email ───────────────────────────────────────────────────────
  const subject = `Beoordeling Vaartraining — ${functie} — ${naam_trainee} (${datumNl})`;

  const bodyText = [
    `Beste,`,
    ``,
    `Bijgaand de beoordeling vaartraining voor:`,
    ``,
    `  Trainee  : ${naam_trainee}`,
    `  Functie  : ${functie}`,
    `  Boot     : ${boot || '—'}`,
    `  Datum    : ${datumNl}`,
    `  Opleider : ${naam_opleider}`,
    `  Conclusie: ${conclusie}`,
    ``,
    isGekwalificeerd
      ? `Omdat de trainee als gekwalificeerd is beoordeeld, is ook de\nVerklaring van Bekwaamheid meegestuurd ter ondertekening.\n`
      : ``,
    `Met vriendelijke groet,`,
    `de Fietsboot`,
  ].join('\n');

  // Convert PDF bytes to base64 for Resend attachments
  const toBase64 = (bytes) => Buffer.from(bytes).toString('base64');

  const beoordelingFilename = sanitiseFilename(
    `Beoordeling_${functie}_${datum_training}_${naam_trainee}.pdf`
  );

  const attachments = [
    {
      filename: beoordelingFilename,
      content:  toBase64(beoordelingBytes),
    },
  ];

  if (isGekwalificeerd && verklaringBytes) {
    attachments.push({
      filename: sanitiseFilename(`Verklaring_${functie}_${naam_trainee}.pdf`),
      content:  toBase64(verklaringBytes),
    });
  }

  // CC list
  const cc = [];
  if (isValidEmail(email_opleider)) cc.push({ email: email_opleider, name: naam_opleider });
  if (isValidEmail(email_trainee))  cc.push({ email: email_trainee,  name: naam_trainee  });

  // ── Send via Resend ───────────────────────────────────────────────────
  const apiKey = process.env.RESEND_API_KEY;
  if (!apiKey) {
    return json({
      ok: false,
      error: 'RESEND_API_KEY niet ingesteld in Netlify omgevingsvariabelen',
    }, 500);
  }

  const resendPayload = {
    from:        `${MAIL_FROM_NAME} <${MAIL_FROM}>`,
    to:          [MAIL_TO],
    cc:          cc.map(c => `${c.name} <${c.email}>`),
    subject,
    text:        bodyText,
    attachments,
  };

  try {
    const resendRes = await fetch('https://api.resend.com/emails', {
      method:  'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type':  'application/json',
      },
      body: JSON.stringify(resendPayload),
    });

    const resendData = await resendRes.json();

    if (!resendRes.ok) {
      console.error('Resend error:', resendData);
      return json({
        ok:    false,
        error: `E-mail fout: ${resendData.message || resendRes.status}`,
      }, 500);
    }
  } catch (err) {
    console.error('Resend fetch error:', err);
    return json({ ok: false, error: `Verbindingsfout: ${err.message}` }, 500);
  }

  // ── Success ───────────────────────────────────────────────────────────
  let msg = `Beoordeling verzonden naar ${MAIL_TO}`;
  if (isGekwalificeerd) msg += ' incl. Verklaring van Bekwaamheid';
  if (cc.length) msg += ` en ${cc.map(c => c.email).join(', ')}`;

  return json({ ok: true, message: msg });
}

// ── Helpers ───────────────────────────────────────────────────────────────
const MAIL_FROM_NAME = 'de Fietsboot';

function json(data, status = 200) {
  return new Response(JSON.stringify(data), {
    status,
    headers: { 'Content-Type': 'application/json', ...corsHeaders() },
  });
}

function corsHeaders() {
  return {
    'Access-Control-Allow-Origin':  '*',
    'Access-Control-Allow-Methods': 'POST, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type',
  };
}

function sanitiseFilename(name) {
  return name.replace(/[^A-Za-z0-9_\-\.]/g, '_');
}

function isValidEmail(email) {
  return typeof email === 'string' && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
