<%*
const slugify = s => s.normalize("NFD").replace(/[\u0300-\u036f]/g, "")
  .toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-+|-+$/g, "");
const q = s => '"' + String(s).replace(/\\/g, "\\\\").replace(/"/g, '\\"') + '"';
const ask = async (label, def = "") => ((await tp.system.prompt(label, def)) ?? "").trim();

// Every folder under english/projects that has an _index.md is a project
const root = app.vault.getAbstractFileByPath("english/projects");
const projects = (root?.children ?? []).filter(f =>
  f.children && f.children.some(c => c.name === "_index.md"));
if (!projects.length) { new Notice("No projects yet: use the New project template first"); return; }

const labels = projects.map(p => {
  const index = p.children.find(c => c.name === "_index.md");
  return app.metadataCache.getFileCache(index)?.frontmatter?.title ?? p.name;
});
const project = await tp.system.suggester(labels, projects, false, "Which project?");
if (!project) { new Notice("Cancelled"); return; }

const title = await ask("Update title");
if (!title) { new Notice("Cancelled: no title given"); return; }

const folder = `${project.path}/${slugify(title)}`;
if (app.vault.getAbstractFileByPath(`${folder}/index.md`)) {
  new Notice(`An update already exists at ${folder}`); return;
}

tR += [
  "---",
  `title: ${q(title)}`,
  `date: ${tp.date.now("YYYY-MM-DDTHH:mm:ssZ")}`,
  "draft: true",
  'description: ""',
  'summary: ""',
  "tags: []",
  '# featured_alt: ""              # if the folder has a featured.* photo',
  '# featured_caption: ""',
  "---",
  "",
  "",
].join("\n");

if (!app.vault.getAbstractFileByPath(folder)) await app.vault.createFolder(folder);
await tp.file.move(`/${folder}/index`);
%>