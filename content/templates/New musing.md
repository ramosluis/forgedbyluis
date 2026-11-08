<%*
const slugify = s => s.normalize("NFD").replace(/[\u0300-\u036f]/g, "")
  .toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-+|-+$/g, "");
const q = s => '"' + String(s).replace(/\\/g, "\\\\").replace(/"/g, '\\"') + '"';
const ask = async (label, def = "") => ((await tp.system.prompt(label, def)) ?? "").trim();

const title = await ask("Musing title");
if (!title) { new Notice("Cancelled: no title given"); return; }

const folder = `english/musings/${slugify(title)}`;
if (app.vault.getAbstractFileByPath(`${folder}/index.md`)) {
  new Notice(`A musing already exists at ${folder}`); return;
}

const tags = (await ask("Tags, comma-separated (optional)"))
  .split(",").map(t => t.trim().toLowerCase()).filter(Boolean);
const alsoIn = await tp.system.suggester(
  ["Only in Musings", "Also list it under Reviews"], [null, "reviews"], false, "Where should it be listed?");

const fm = [
  "---",
  `title: ${q(title)}`,
  `date: ${tp.date.now("YYYY-MM-DDTHH:mm:ssZ")}`,
  "draft: true",
  'description: ""',
  'summary: ""',
  tags.length ? "tags:\n" + tags.map(t => `    - ${q(t)}`).join("\n") : "tags: []",
];
if (alsoIn) fm.push(`show_in: ["${alsoIn}"]`);
fm.push('# featured_alt: ""              # if the folder has a featured.* photo',
        '# featured_caption: ""', "---", "", "");
tR += fm.join("\n");

if (!app.vault.getAbstractFileByPath(folder)) await app.vault.createFolder(folder);
await tp.file.move(`/${folder}/index`);
%>