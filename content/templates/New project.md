<%*
const slugify = s => s.normalize("NFD").replace(/[\u0300-\u036f]/g, "")
  .toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-+|-+$/g, "");
const q = s => '"' + String(s).replace(/\\/g, "\\\\").replace(/"/g, '\\"') + '"';
const ask = async (label, def = "") => ((await tp.system.prompt(label, def)) ?? "").trim();

const title = await ask("Project name (e.g. 3D printed R2-D2)");
if (!title) { new Notice("Cancelled: no name given"); return; }
const slug = await ask("Folder name for the URL", slugify(title));

const folder = `english/projects/${slug}`;
if (app.vault.getAbstractFileByPath(`${folder}/_index.md`)) {
  new Notice(`A project already exists at ${folder}`); return;
}

const description = await ask("One line on the project's goal");
const status = await tp.system.suggester(
  ["Planning", "In progress", "Paused", "Done"],
  ["Planning", "In progress", "Paused", "Done"], false, "Status") ?? "Planning";

tR += [
  "---",
  `title: ${q(title)}`,
  `description: ${q(description)}`,
  `status: ${q(status)}`,
  "---",
  "",
  "",
].join("\n");

if (!app.vault.getAbstractFileByPath(folder)) await app.vault.createFolder(folder);
await tp.file.move(`/${folder}/_index`);
%>