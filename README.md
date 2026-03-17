# TuneBot
Piano Chord Progression Generator
// tunebot.js — corrected full version
// No external libs required. Works with your index.html layout.

// ---------- Utility data ----------
const SEMITONE_NAMES_SHARP = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"];
// Map flats to equivalent sharps for internal consistency
const FLAT_TO_SHARP = {
  Db: "C#",
  Eb: "D#",
  Fb: "E",
  Gb: "F#",
  Ab: "G#",
  Bb: "A#",
  Cb: "B"
};

// Interval formulas (in semitones from root)
const CHORD_FORMULAS = {
  "": [0,4,7],            // major triad
  "maj": [0,4,7],
  "m": [0,3,7],           // minor triad
  "5": [0,7],             // power chord
  "dim": [0,3,6],
  "aug": [0,4,8],
  "7": [0,4,7,10],        // dominant 7
  "maj7": [0,4,7,11],
  "m7": [0,3,7,10],
  "m7b5": [0,3,6,10],
  "sus2": [0,2,7],
  "sus4": [0,5,7],
  "add9": [0,4,7,14],
  "9": [0,4,7,10,14],
  "11": [0,4,7,10,14,17],
  "13": [0,4,7,10,14,21],
};

// ---------- Progression & genre data (keeps your generator behaviour) ----------
const progressionPatterns = [
  [0,3,4],
  [0,5,1,4],
  [0,2,3,4],
  [0,4,5,3],
  [0,3,5,4],
  [1,4,0],
  [0,1,5,4],
  [0,4,6,0],
  [2,5,1,4],
  [0,6,0],
  [0,4,0,4],
  [4,3,0,1],
  [5,4,3,0],
  [0,3,4,1],
  [0,2,5,3],
  [0,3,6,0],
  [0,5,3,1],
  [5,1,0,4],
  [0,4,5,3],
  [1,3,4,0]
];

const majorKey = {
  "C": ["C", "Dm", "Em", "F", "G", "Am", "Bdim"],
  "G": ["G", "Am", "Bm", "C", "D", "Em", "F#dim"],
  "D": ["D", "Em", "F#m", "G", "A", "Bm", "C#dim"],
  "A": ["A", "Bm", "C#m", "D", "E", "F#m", "G#dim"],
  "E": ["E", "F#m", "G#m", "A", "B", "C#m", "D#dim"],
  "B": ["B", "C#m", "D#m", "E", "F#", "G#m", "A#dim"],
  "F#": ["F#", "G#m", "A#m", "B", "C#", "D#m", "E#dim"],
  "C#": ["C#", "D#m", "E#m", "F#", "G#", "A#m", "B#dim"],
  "G#": ["G#", "A#m", "Cm", "C#", "D#", "Fm", "Gdim"],
  "D#": ["D#", "Fm", "Gm", "G#", "A#", "Cm", "Ddim"],
  "A#": ["A#", "Cm", "Dm", "D#", "F", "Gm", "Am7b5"],
  "F": ["F", "Gm", "Am", "A#", "C", "Dm", "Edim"]
};

const majorKeySevenths = {
  "C": ["Cmaj7", "Dm7", "Em7", "Fmaj7", "G7", "Am7", "Bm7b5"],
  "G": ["Gmaj7", "Am7", "Bm7", "Cmaj7", "D7", "Em7", "F#m7b5"],
  "D": ["Dmaj7", "Em7", "F#m7", "Gmaj7", "A7", "Bm7", "C#m7b5"],
  "A": ["Amaj7", "Bm7", "C#m7", "Dmaj7", "E7", "F#m7", "G#m7b5"],
  "E": ["Emaj7", "F#m7", "G#m7", "Amaj7", "B7", "C#m7", "D#m7b5"],
  "B": ["Bmaj7", "C#m7", "D#m7", "Emaj7", "F#7", "G#m7", "A#m7b5"],
  "F#": ["F#maj7", "G#m7", "A#m7", "Bmaj7", "C#7", "D#m7", "E#m7b5"],
  "C#": ["C#maj7", "D#m7", "E#m7", "F#maj7", "G#7", "A#m7", "B#m7b5"],
  "G#": ["G#maj7", "A#m7", "Cm7", "C#maj7", "D#7", "Fm7", "Gm7b5"],
  "D#": ["D#maj7", "Fm7", "Gm7", "G#maj7", "A#7", "Cm7", "Dm7b5"],
  "A#": ["A#maj7", "Cm7", "Dm7", "D#maj7", "F7", "Gm7", "Am7b5"],
  "F": ["Fmaj7", "Gm7", "Am7", "A#maj7", "C7", "Dm7", "Em7b5"]
};

// genre -> probability of per-chord being a 7th variant (0..1)
const genreSeventhChance = {
  Pop: 0.2,
  Rock: 0.1,
  Classical: 0.05,
  Jazz: 0.8,
  "R&B": 0.6,
  "Lo-Fi": 0.65
};

// ---------- Note parsing helpers ----------
function normalizeRootName(input) {
  // Accept roots like "Bb", "C#", "Cb", "E#", "Gb"
  if (!input) return null;
  // Uppercase letter plus optional accidental
  const m = input.match(/^([A-Ga-g])([#b]?)/);
  if (!m) return null;
  let name = m[1].toUpperCase() + (m[2] || "");
  // map flats to sharps for internal consistency
  if (name.length === 2 && name[1] === "b") {
    const flat = name;
    if (FLAT_TO_SHARP[flat]) return FLAT_TO_SHARP[flat];
    // fallback: convert using semitone table
    const idx = SEMITONE_NAMES_SHARP.indexOf(name[0]);
    return SEMITONE_NAMES_SHARP[(idx + 11) % 12];
  }
  // map rare E# / B# etc
  if (FLAT_TO_SHARP[name]) return FLAT_TO_SHARP[name];
  return name;
}

function parseChordName(chordName) {
  // Expect examples: C, Cm, Cmaj7, C#m7, Dbmaj7, Bbm7b5
  // Regex: root (A-G with optional #/b), then everything else is type
  const m = chordName.match(/^([A-Ga-g][#b]?)(.*)$/);
  if (!m) return { root: null, type: "" };
  const rootRaw = m[1];
  const typeRaw = m[2] || "";
  const root = normalizeRootName(rootRaw);
  let type = typeRaw.trim();

  // normalize some type shorthands
  // ensure order of replacement: e.g. 'm7b5' before 'm7' before 'm'
  if (/m7b5/i.test(type)) type = "m7b5";
  else if (/m7/i.test(type) && !/maj/i.test(type)) type = "m7";
  else if (/maj7/i.test(type) || /^M7$/i.test(type)) type = "maj7";
  else if (/^maj$/i.test(type)) type = "maj";
  else if (/^m$/i.test(type)) type = "m";
  else if (/^dim/i.test(type)) type = "dim";
  else if (/^aug/i.test(type)) type = "aug";
  else if (/^sus2/i.test(type)) type = "sus2";
  else if (/^sus4/i.test(type)) type = "sus4";
  else if (/^add9/i.test(type)) type = "add9";
  else if (/^9($|[^a-zA-Z])/i.test(type) || /^9$/i.test(type)) type = "9";
  else if (/^11/i.test(type)) type = "11";
  else if (/^13/i.test(type)) type = "13";
  else if (/^7$/i.test(type) || /^dom7$/i.test(type)) type = "7";
  // default to "" meaning major triad
  if (type === "") type = "";

  return { root, type };
}

// returns array of note names (sharp names, octave-less) e.g. ['C','E','G','B']
// sorted ascending from the root (no inversions)
function chordToNotes(chordName) {
  const { root, type } = parseChordName(chordName);
  if (!root) return [];

  const formula = CHORD_FORMULAS[type] || CHORD_FORMULAS[""];
  const rootIndex = SEMITONE_NAMES_SHARP.indexOf(root);
  if (rootIndex === -1) return [];

  // collect note names (may include duplicates for odd chords) and dedupe preserving order
  const raw = formula.map(interval => SEMITONE_NAMES_SHARP[(rootIndex + interval) % 12]);
  const unique = [];
  for (const n of raw) if (!unique.includes(n)) unique.push(n);

  // sort ascending by semitone distance from root (0 = root, then increasing)
  unique.sort((a, b) => {
    const aPos = (SEMITONE_NAMES_SHARP.indexOf(a) - rootIndex + 12) % 12;
    const bPos = (SEMITONE_NAMES_SHARP.indexOf(b) - rootIndex + 12) % 12;
    return aPos - bPos;
  });

  // ensure root is first (safety)
  if (unique[0] !== root) {
    const rootIdx = unique.indexOf(root);
    if (rootIdx > -1) {
      unique.splice(rootIdx, 1);
      unique.unshift(root);
    } else {
      unique.unshift(root);
    }
  }

  return unique;
}

// ---------- Piano DOM builder (C4..B5, correct black key placement) ----------

function createMiniPiano(activeNotes = []) {
  const WHITE_ORDER = ["C","D","E","F","G","A","B"];
  const BLACK_AFTER = { C: "C#", D: "D#", F: "F#", G: "G#", A: "A#" };

  // Normalize notes (remove octave, convert flats to sharps)
  const activeNorm = activeNotes.map(n => {
    const cleaned = String(n).replace(/\d/g,"");
    return FLAT_TO_SHARP[cleaned] || cleaned;
  });

  const piano = document.createElement("div");
  piano.className = "mini-piano";

  // Determine starting octave so root is the first visible key
  const rootNote = activeNorm[0];
  let startOctave = 4; // default
  if (rootNote) {
    const rootIndex = WHITE_ORDER.indexOf(rootNote[0]);
    // Start one octave below if root is high letter (so chord fits)
    startOctave += Math.floor(rootIndex / WHITE_ORDER.length);
  }

  const whitesContainer = document.createElement("div");
  whitesContainer.style.display = "inline-block";
  const whiteKeyElems = [];

  // Build two octaves starting from root
  for (let o = 0; o < 2; o++) {
    for (const w of WHITE_ORDER) {
      const full = w + (startOctave + o);
      const wk = document.createElement("div");
      wk.className = "white-key";
      wk.dataset.note = full;
      whitesContainer.appendChild(wk);
      whiteKeyElems.push(wk);
    }
  }

  piano.appendChild(whitesContainer);

  const whiteW = parseInt(getComputedStyle(document.documentElement).getPropertyValue('--white-w')) || 28;
  const blackW = parseInt(getComputedStyle(document.documentElement).getPropertyValue('--black-w')) || 18;

  // Build black keys
  for (let idx = 0; idx < whiteKeyElems.length; idx++) {
    const whiteNoteFull = whiteKeyElems[idx].dataset.note;
    const base = whiteNoteFull.replace(/\d/g,"");
    const octave = parseInt(whiteNoteFull.match(/\d+/)[0],10);
    if (BLACK_AFTER[base]) {
      const blackName = BLACK_AFTER[base];
      const fullBlack = blackName + octave;
      const bk = document.createElement("div");
      bk.className = "black-key";
      bk.dataset.note = fullBlack;

      const left = (idx * whiteW) + (whiteW - Math.round(blackW/2));
      bk.style.left = left + "px";

      piano.appendChild(bk);
    }
  }

// --- Highlight notes so they appear starting from the root and to the right ---
const allKeys = [...piano.querySelectorAll(".white-key"), ...piano.querySelectorAll(".black-key")];

// Sort active notes based on their semitone distance from the root
const rootPos = SEMITONE_NAMES_SHARP.indexOf(rootNote);
const sortedNotes = activeNorm.slice().sort((a, b) => {
  const aPos = (SEMITONE_NAMES_SHARP.indexOf(a) - rootPos + 12) % 12;
  const bPos = (SEMITONE_NAMES_SHARP.indexOf(b) - rootPos + 12) % 12;
  return aPos - bPos;
});

// Find the exact key (white or black) that matches the root
const rootKey = Array.from(allKeys).find(k => {
  const n = k.dataset.note.replace(/\d/g, "");
  return n === rootNote;
});

// If found, set it as starting index; otherwise, start at 0
const startIndex = rootKey ? Array.from(allKeys).indexOf(rootKey) : 0;

// Highlight notes in sorted order starting from root
sortedNotes.forEach(note => {
  for (let j = startIndex; j < allKeys.length; j++) {
    const keyName = allKeys[j].dataset.note.replace(/\d/g, "");
    if (keyName === note && !allKeys[j].dataset.active) {
      allKeys[j].dataset.active = "true";
      break;
    }
  }
});

  return piano;
}

// ---------- Generator + render logic ----------
function resetUI(){
  document.getElementById("results").innerHTML = "";
  document.getElementById("progressionLabel").textContent = "Select a scale and genre to start 🎶";
}

function displayProgression(selectedKey, selectedGenre) {
  const triads = majorKey[selectedKey];
  const sevenths = majorKeySevenths[selectedKey];
  if (!triads) {
    document.getElementById("progressionLabel").textContent = "Unsupported key";
    return;
  }

  const seventhChance = genreSeventhChance[selectedGenre] ?? 0.2;
  const pattern = progressionPatterns[Math.floor(Math.random()*progressionPatterns.length)];

  // clear
  const results = document.getElementById("results");
  results.innerHTML = "";

  const chordNames = [];

  // show up to pattern.length chords (limit 4 to fit UI if you prefer)
  const count = Math.min(pattern.length, 4);
  for (let i=0;i<count;i++){
    const degreeIndex = pattern[i];
    const use7 = Math.random() < seventhChance;
    const chordName = use7 ? (sevenths[degreeIndex] || triads[degreeIndex]) : triads[degreeIndex];

    chordNames.push(chordName);

    // compute notes (without octave)
    const notes = chordToNotes(chordName);

    // build card
    const card = document.createElement("div");
    card.className = "chord-card";

    const info = document.createElement("div");
    info.className = "chord-info";
    const nameEl = document.createElement("div");
    nameEl.className = "chord-name";
    nameEl.textContent = chordName;
    const meta = document.createElement("div");
    meta.className = "chord-meta";
    meta.textContent = notes.join(" • ");

    info.appendChild(nameEl);
    info.appendChild(meta);

    const piano = createMiniPiano(notes);

    card.appendChild(info);
    card.appendChild(piano);
    results.appendChild(card);
  }

  document.getElementById("progressionLabel").textContent = chordNames.join(" — ");
}

// ---------- UI wiring ----------
document.getElementById("generateBtn").addEventListener("click", () => {
  const selectedKey = document.getElementById("keyDropdown").value;
  const selectedGenre = document.getElementById("genreDropdown").value;
  if (selectedKey === "Select Scale") {
    document.getElementById("progressionLabel").textContent = "Please select a scale!";
    return;
  }
  if (selectedGenre === "Select Genre") {
    document.getElementById("progressionLabel").textContent = "Please select a genre!";
    return;
  }
  displayProgression(selectedKey, selectedGenre);
});

document.getElementById("keyDropdown").addEventListener("change", () => {
  if (document.getElementById("keyDropdown").value === "Select Scale") resetUI();
});
document.getElementById("genreDropdown").addEventListener("change", () => {
  if (document.getElementById("genreDropdown").value === "Select Genre") resetUI();
});

// initialize UI
resetUI();
