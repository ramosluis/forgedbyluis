<%*
// ---------- helpers ----------
const slugify = s => s.normalize("NFD").replace(/[\u0300-\u036f]/g, "")
  .toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-+|-+$/g, "");
const q = s => '"' + String(s).replace(/\\/g, "\\\\").replace(/"/g, '\\"') + '"';
const ask = async (label, def = "") => ((await tp.system.prompt(label, def)) ?? "").trim();

// ---------- questions ----------
const title = await ask("Book title");
if (!title) { new Notice("Cancelled: no title given"); return; }

const folder = `english/reviews/${slugify(title)}`;
if (app.vault.getAbstractFileByPath(`${folder}/index.md`)) {
  new Notice(`A review already exists at ${folder}`); return;
}

const author = await ask("Author");
const rating = await tp.system.suggester(
  ["★★★★★  5", "★★★★☆  4", "★★★☆☆  3", "★★☆☆☆  2", "★☆☆☆☆  1", "DNF", "No rating yet"],
  [5, 4, 3, 2, 1, 0, null], false, "Rating");
let lang = await tp.system.suggester(
  ["English", "Spanish", "Other…"], ["English", "Spanish", "other"], false, "Original language");
if (lang === "other") lang = await ask("Original language");
const tags = (await ask("Genres, comma-separated (e.g. fantasy, horror)"))
  .split(",").map(t => t.trim().toLowerCase()).filter(Boolean);
const series = await ask("Series (leave empty if none)");
const seriesOrder = series ? await ask(`Number in ${series}`) : "";
const subseries = series ? await ask("Sub-series (leave empty if none)") : "";
const subOrder = subseries ? await ask(`Number in ${subseries}`) : "";
const goodreads = await ask("Goodreads URL (optional)");

// ---------- front matter ----------
const fm = [
  "---",
  `title: ${q(title)}`,
  `date: ${tp.date.now("YYYY-MM-DDTHH:mm:ssZ")}`,
  "draft: true",
  'description: ""',
  'summary: ""',
  tags.length ? "tags:\n" + tags.map(t => `    - ${q(t)}`).join("\n") : "tags: []",
];
if (series) {
  fm.push("series:", `    - ${q(series)}`);
  if (seriesOrder) fm.push(`series_order: ${seriesOrder}`);
}
if (subseries) {
  fm.push("subseries:", `    - ${q(subseries)}`);
  if (subOrder) fm.push(`subseries_order: ${subOrder}`);
}
fm.push("book:", `    author: ${q(author)}`);
if (rating !== null && rating !== undefined) fm.push(`    rating: ${rating}`);
else fm.push("    # rating: 3                    # 1-5, or 0 for DNF");
if (lang && lang !== "English") fm.push(`    original_language: ${q(lang)}`);
if (goodreads) fm.push(`    goodreads: ${q(goodreads)}`);
fm.push("---", "", "");
tR += fm.join("\n");

// ---------- move into place ----------
if (!app.vault.getAbstractFileByPath(folder)) await app.vault.createFolder(folder);
await tp.file.move(`/${folder}/index`);
%>