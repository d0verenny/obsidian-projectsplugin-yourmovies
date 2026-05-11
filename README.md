# obsidian-projectsplugin-yourmovies
My script to use with Obisidian Projects plugin along with MediaDB plugin to have your own database of your games and movies!

Hi! Saw your post on obsidian forums regarding the plugin and replied to you there :) Are you going to work on the project?

I also have something to share with you. 

The killer feature of Projects is that unlike Bases core plugin, it allows running the dataview query from the YAML/properties. Which you can mention to attract more people. Not everybody knows that. 

I will diplicate the info from [this post](https://forum.obsidian.md/t/development-for-projects-plugin-needed/101089/27) on Obsidian forums and add more details for your testing purposes.

# Projects dataviewjs query - the overview

I used obsidian-projects to make my backlog database of movies and games. I use the **Media DB** plugin to grab games/movies info from TMDB, Steam, etc.

To make it output my reviews from Media DB created notes, I wrote a dataviewjs script and put it in the frontmatter of the Media DB plugin's template note. It fetches the text from the body of the note, between the delimiters "## Review" and "***" (a straight line).

This is how a note created by Media DB looks like:

<img width="1000" height="1032" alt="image" src="https://github.com/user-attachments/assets/8155cde1-5188-45a8-9411-351df5f8526b" />

---

So, when I open my Projects page, I can see my review from that note:

<img width="1024" height="1032" alt="YETI6fG7As" src="https://github.com/user-attachments/assets/a6dfce2e-48fd-4f7f-860a-3a782808e2c6" />


---

## How to set up

#### Prerequisites:

- **Obsidian** (obviously)
- **"Projects"** plugin (obviously)
- **YouTube API key** (lots of guides out there on how to get it)
- **Media DB plugin** — to fetch video game or movies and make notes
- (optional, but preferable) **Extended Media plugin** — a better videoplayer for youtube (at a date, yt embeds don't work in Obsidian ver. 1.9.14)

---

## Step 1. Set up Media DB plugin

### 1. Create a template note for Media DB

Below is the template note with the dataviewjs query code + YouTube embed code. You may called it **MediaDBGames.md**

❗ Please note that you have to enter your YouTube API key in the `let apiKey = "E "` section

```
---
type: 
subType: ""
title: 
englishTitle: 
year: ""
dataSource: 
url: 
id: 
developers: 
publishers: 
genres: 
onlineRating: 
image: "![]({{ image }})"
released: 
releaseDate: 
played: 
personalRating: 
dataviewCode: |
  ```dataviewjs
  // === Wait for the frontmatter to populate ===
  await new Promise(resolve => setTimeout(resolve, 500));

  let noteName = tp.frontmatter.englishTitle;
  let notePath = `Media DB/games/${noteName}.md`;

  let heading = "## Review";
  let delimiter = "***";

  let file = app.vault.getAbstractFileByPath(notePath);

  if (!file) {
      dv.paragraph("_File not found:_ " + notePath);
  } else {
      (async () => {
          let content = await app.vault.read(file);
          let lines = content.split(/\r?\n/);

          let inSection = false;
          let sectionLines = [];

          for (let line of lines) {
              if (!inSection) {
                  if (line.trim() === heading) inSection = true;
              } else {
                  if (line.trim() === delimiter) break;
                  sectionLines.push(line);
              }
          }

          if (sectionLines.length) {
              dv.paragraph(sectionLines.join("\n").trim());
          } else {
              dv.paragraph("_No " + heading + " section found in note:_ " + notePath);
          }
      })();
  }
tags:
---

<%* 
await new Promise(resolve => setTimeout(resolve, 1000)); // wait 0.5s

let headingTitle = tp.frontmatter.englishTitle; // takes the name from frontmatter englishTitle
tR += `# ${headingTitle}`;
%>

***

videoUrl::![](<%*
await new Promise(resolve => setTimeout(resolve, 1000)); // wait 0.5s

let ytTitle = tp.frontmatter.englishTitle; // YouTube title. takes the name from frontmatter englishTitle
let query = encodeURIComponent(ytTitle);

let apiKey = "ENTER_YOUR_YOUTUBE_API_KEY!";
let url = `https://www.googleapis.com/youtube/v3/search?part=snippet&type=video&q=${query}&maxResults=1&key=${apiKey}`;

let response = await fetch(url);
let data = await response.json();

let vidId = data.items?.[0]?.id?.videoId ?? "";
tR += `https://www.youtube.com/watch?v=${vidId}`;
%>)

***

## Review

Check check

***

```dataviewjs
// === Wait for the note title to be ready (0.5s) ===
await new Promise(resolve => setTimeout(resolve, 1000));

let noteName = tp.frontmatter.englishTitle; 
let notePath = `Media DB/games/${noteName}.md`;

let heading = "## Review"; 
let delimiter = "***";

let file = app.vault.getAbstractFileByPath(notePath);

if (!file) {
  dv.paragraph("_File not found:_ " + notePath);
} else {
  (async () => {
	  let content = await app.vault.read(file);
	  let lines = content.split(/\r?\n/);

	  let inSection = false;
	  let sectionLines = [];

	  for (let line of lines) {
		  if (!inSection) {
			  if (line.trim() === heading) inSection = true;
		  } else {
			  if (line.trim() === delimiter) break;
			  sectionLines.push(line);
		  }
	  }

	  if (sectionLines.length) {
		  dv.paragraph(sectionLines.join("\n").trim());
	  } else {
		  dv.paragraph("_No " + heading + " section found in note:_ " + notePath);
	  }
  })();
}
```

#### What the script does exactly?

##### 1. Searching for youtube video based on the englishTitle in the front matter and pastes it in the embeded format `![](youtube.com/yourvideo`). 

If you want to change it to something else, for example, the file name, change this variable `let ytTitle = tp.frontmatter.englishTitle;` it to `let ytTitle = tp.frontmatter.title` or `tp.file.title` (not sure if the latter one will work, but you can try.

##### 2. Makes a dataviewjs query from the body of the document, between delimiters `## Review` and `***` (straight line).

**Why not `---`?**

To avoid any conflicts. It's cleaner that way. You can change delimiters to your custom ones in the code. Just change them in these lines after `=`:

`let heading = "## Review"; `
`let delimiter = "***";`

---

### 2. Get the OMDB Api key and enter
### 3. Get the Giant Bomb API key and enter
### 4. Enable Template Integration and, optionally - "Open in new tab"

<img width="1000" height="882" alt="image" src="https://github.com/user-attachments/assets/34a00c5b-aad7-4fa4-b363-de4cfc7c9a17" />

### 5. Put the path to your template note.

<img width="1000" height="880" alt="image" src="https://github.com/user-attachments/assets/55b75dfc-fa3f-48ae-8b83-6fd0dd807eb6" />

### 6. Create a MediaDB note via Command Palete. The command:

`Media DB: Create Media DB Entry (Game)"

### 7. Search and select a game

<img width="514" height="463" alt="image" src="https://github.com/user-attachments/assets/3f97fefa-4fa5-4c5b-a2fa-2018d6a0559a" />

<img width="510" height="882" alt="Obsidian_p9ftbPHztX" src="https://github.com/user-attachments/assets/9c701d18-cd94-48c2-92a8-b1fcc832b64a" />

<img width="529" height="742" alt="Obsidian_ss2zpWpHDE" src="https://github.com/user-attachments/assets/e3f34066-d904-4ba1-8f1b-bc3a2775464d" />

Ignore the error about Moby Games API. We didn't set it, so it appears.

### 8. In the created note, write your review for testing purposes.

<img width="1024" height="1032" alt="image" src="https://github.com/user-attachments/assets/0775d9fd-361a-484e-924b-0fd14bfa7baf" />

---

## Step 2. Set up the Projects plugin

1. Create a new project (or edit the existing one: click 3 dots -> "Edit project")). Give it a name.
3. Set **Data Source** to `Dataview`.

Put this query in the open field:

```
TABLE
    title AS "Title",
    videoUrl AS "Video",
    englishTitle AS "English Title",
    year AS "Year",
    dataSource AS "Data Source",
    year AS "Year",
    dataSource AS "Data Source",
    url AS "Steam Page",
    id AS "Steam ID",
    developers AS "Developers",
    publishers AS "Publishers",
    genres AS "Genres",
    onlineRating AS "Online Rating",
    image AS "Image",
    released AS "Released?",
    releaseDate AS "Release Date",
    played AS "Played?",
    personalRating AS "My Rating",
    subType AS "Sub-Type",
    dataviewCode as "Review"
FROM "Media DB/games"
WHERE type = "game"
SORT releaseDate DESC
```
<img width="518" height="477" alt="image" src="https://github.com/user-attachments/assets/3e8be116-7f1e-4117-8a7a-f43358647eb5" />

--- 
### 3. In the created project, you will see the fields. Scroll to the right to the Review field. Click 3 dots -> Configure field

<img width="1000" height="582" alt="image" src="https://github.com/user-attachments/assets/7f8fe481-45bd-409a-8e99-9e6042c15370" />

### 4. In the open window, enable Rich formatting

<img width="519" height="366" alt="image" src="https://github.com/user-attachments/assets/11c575fd-8aaa-464f-9343-b98e92c6790b" />

### 5. Do the same with Video, Steam Page, and Image fields (the latter one is optional). 

❗When enabling Rich format, the dataviewjs code will be able to launch. And embedded YouTube videos will show up in the Gallery view tab.

---

### 6. Create the gallery view tab (click + in the middle)

<img width="994" height="411" alt="image" src="https://github.com/user-attachments/assets/5c89a6eb-ae48-47eb-af6a-78210f04dd07" />

<img width="517" height="268" alt="image" src="https://github.com/user-attachments/assets/68667228-5e0f-4e34-9a27-a90a9d92f629" />

---

### 7. Set Cover to Image property. This will make the covers show.

<img width="927" height="356" alt="image" src="https://github.com/user-attachments/assets/01a627e3-aa17-4b8d-995b-b6aab4d4ef3f" />

---

### 8. Set Include Fields to your liking. Add Review field.

---

### 9. Check if the Review and the video show up in the fields.

❗The error 153 on YT video is expected on v. 1.9.14. Wait for Obsidian devs to fix it.

<img width="1024" height="1032" alt="image" src="https://github.com/user-attachments/assets/256c9d2b-947a-490a-98f2-8c3a7fc6e505" />

---

## Step 3. Thats it!

Pls, let me know what you think!

---

## Important Notes:

_Originally posted by @d0verenny in https://github.com/Acylation/obsidian-projects/discussions/1_
