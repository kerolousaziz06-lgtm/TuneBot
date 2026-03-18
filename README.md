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
