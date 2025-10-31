/* Looker Studio Community Visualization: Safety Wheel Donut
 * Required fields: Dimension 1 = Date, Dimension 2 = Status, Metric = Value
 */
const dscc = require('dscc');

const DEFAULTS = {
  safeColor: '#2ECC71',
  warningColor: '#8E6B3A',
  unsafeColor: '#E74C3C',
  hole: 0.65,
  showLegend: false,
};

function getOption(style, key) {
  return (style && style[key] && (style[key].value !== undefined))
    ? style[key].value
    : DEFAULTS[key];
}

function draw(data) {
  const style = data.style || {};
  const container = document.getElementById('safety-wheel-root');
  container.innerHTML = '';

  const width = container.clientWidth || 400;
  const height = container.clientHeight || 400;
  const radius = Math.min(width, height) / 2;
  const innerRadius = radius * getOption(style, 'hole');

  // Build rows: [{date, status, value}...]
  const rows = (data.tables.DEFAULT || []).map(r => ({
    date: r.dimID0,
    status: r.dimID1,
    value: +r.metricID0 || 0
  }));

  // Sort by date string to keep stable order
  rows.sort((a, b) => ('' + a.date).localeCompare('' + b.date));

  // Map statuses to colors
  const colors = {
    'safe': getOption(style, 'safeColor'),
    'warning': getOption(style, 'warningColor'),
    'unsafe': getOption(style, 'unsafeColor')
  };

  const total = rows.reduce((s, r) => s + (r.value || 0), 0) || 1;

  // Create SVG
  const svgNS = 'http://www.w3.org/2000/svg';
  const svg = document.createElementNS(svgNS, 'svg');
  svg.setAttribute('width', width);
  svg.setAttribute('height', height);
  svg.style.display = 'block';

  const g = document.createElementNS(svgNS, 'g');
  g.setAttribute('transform', `translate(${width/2},${height/2})`);
  svg.appendChild(g);

  // draw slices
  let angle = -Math.PI / 2; // start at top
  rows.forEach(r => {
    const sliceAngle = (r.value / total) * Math.PI * 2;
    const start = angle;
    const end = angle + sliceAngle;
    angle = end;

    const outerStart = polarToCartesian(radius, start);
    const outerEnd = polarToCartesian(radius, end);
    const innerStart = polarToCartesian(innerRadius, end);
    const innerEnd = polarToCartesian(innerRadius, start);
    const largeArc = sliceAngle > Math.PI ? 1 : 0;

    const pathData = [
      'M', outerStart.x, outerStart.y,
      'A', radius, radius, 0, largeArc, 1, outerEnd.x, outerEnd.y,
      'L', innerStart.x, innerStart.y,
      'A', innerRadius, innerRadius, 0, largeArc, 0, innerEnd.x, innerEnd.y,
      'Z'
    ].join(' ');

    const path = document.createElementNS(svgNS, 'path');
    const key = ('' + r.status).trim().toLowerCase();
    path.setAttribute('d', pathData);
    path.setAttribute('fill', colors[key] || '#BDC3C7');
    path.setAttribute('stroke', 'white');
    path.setAttribute('stroke-width', '1');
    path.setAttribute('data-date', r.date);
    path.setAttribute('data-status', r.status);
    g.appendChild(path);
  });

  // center label optional (add an S)
  const text = document.createElementNS(svgNS, 'text');
  text.setAttribute('x', 0);
  text.setAttribute('y', 6);
  text.setAttribute('text-anchor', 'middle');
  text.setAttribute('font-size', Math.round(innerRadius * 0.6));
  text.setAttribute('fill', '#3A3A3A');
  text.textContent = 'S';
  g.appendChild(text);

  container.appendChild(svg);

  // optional legend
  if (getOption(style, 'showLegend')) {
    const legend = document.createElement('div');
    legend.style.marginTop = '8px';
    Object.entries(colors).forEach(([k, c]) => {
      const row = document.createElement('div');
      row.style.display = 'inline-flex';
      row.style.alignItems = 'center';
      row.style.marginRight = '12px';
      const sw = document.createElement('span');
      sw.style.display = 'inline-block';
      sw.style.width = '10px';
      sw.style.height = '10px';
      sw.style.marginRight = '6px';
      sw.style.background = c;
      sw.style.border = '1px solid #ccc';
      row.appendChild(sw);
      row.appendChild(document.createTextNode(k.charAt(0).toUpperCase() + k.slice(1)));
      legend.appendChild(row);
    });
    container.appendChild(legend);
  }
}

function polarToCartesian(r, angle) {
  return { x: r * Math.cos(angle), y: r * Math.sin(angle) };
}

function onDraw(data) {
  // Create root container once
  let root = document.getElementById('safety-wheel-root');
  if (!root) {
    root = document.createElement('div');
    root.id = 'safety-wheel-root';
    root.style.width = '100%';
    root.style.height = '100%';
    document.body.appendChild(root);
  }
  draw(data);
}

dscc.subscribeToData(onDraw, {transform: dscc.objectTransform});
