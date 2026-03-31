// netlify/functions/generate-pdf.mjs
// Called by the "Download PDF" button in vaartraining.html
// Returns the PDF as a binary response for direct browser download.

import { buildBeoordelingPdf } from './pdf-builder.mjs';

export default async function handler(req) {

  if (req.method === 'OPTIONS') {
    return new Response(null, { status: 204, headers: corsHeaders() });
  }

  if (req.method !== 'POST') {
    return new Response('POST only', { status: 405 });
  }

  let data;
  try {
    data = await req.json();
  } catch {
    return new Response('Invalid JSON', { status: 400 });
  }

  try {
    const pdfBytes = await buildBeoordelingPdf(data);
    const datum    = data.datum_training || '';
    const naam     = (data.naam_trainee || '').replace(/\s+/g, '_');
    const functie  = (data.functie || '').replace(/\s+/g, '_');
    const filename = `Beoordeling_${functie}_${datum}_${naam}.pdf`;

    return new Response(pdfBytes, {
      status: 200,
      headers: {
        'Content-Type':        'application/pdf',
        'Content-Disposition': `attachment; filename="${filename}"`,
        ...corsHeaders(),
      },
    });
  } catch (err) {
    console.error('PDF error:', err);
    return new Response(JSON.stringify({ ok: false, error: err.message }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' },
    });
  }
}

function corsHeaders() {
  return {
    'Access-Control-Allow-Origin':  '*',
    'Access-Control-Allow-Methods': 'POST, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type',
  };
}
