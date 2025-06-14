<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Modern Audio Stream Player</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0;
    font-family: 'Inter', sans-serif;
    background: #111;
    color: #fff;
  }
  .app {
    display: flex;
    flex-direction: column;
    height: 100vh;
  }
  .library {
    flex: 1;
    overflow-y: auto;
    padding: 20px;
  }
  .song-item {
    background: #222;
    border-radius: 12px;
    padding: 16px;
    margin-bottom: 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
    transition: background 0.3s;
  }
  .song-item:hover {
    background: #333;
  }
  .song-info {
    display: flex;
    flex-direction: column;
  }
  .song-title {
    font-weight: 600;
  }
  .player {
    background: #1e1e1e;
    padding: 20px;
    box-shadow: 0 -2px 10px rgba(0,0,0,0.3);
  }
  .info, .controls, .progress {
    margin: 10px 0;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .controls button {
    background: #333;
    border: none;
    border-radius: 8px;
    padding: 10px;
    color: #fff;
    cursor: pointer;
    font-size: 16px;
    transition: background 0.3s;
  }
  .controls button:hover {
    background: #555;
  }
  input[type=range] {
    flex: 1;
    margin: 0 10px;
  }
</style>
</head>
<body>
<div class="app">
  <div class="library" id="library"></div>
  <div class="player">
    <div class="info">
      <span id="current-title">Title</span> — <span id="current-artist">Artist</span>
    </div>
    <div class="controls">
      <button onclick="prevSong()">⏮️</button>
      <button onclick="togglePlayPause()" id="play-btn">▶️</button>
      <button onclick="nextSong()">⏭️</button>
      <button onclick="toggleLoop()">🔁</button>
      <button onclick="toggleShuffle()">🔀</button>
    </div>
    <div class="progress">
      <span id="current-time">0:00</span>
      <input type="range" id="progress-bar" value="0" onchange="seekAudio(this.value)">
      <span id="duration">0:00</span>
    </div>
  </div>
</div>
<audio id="audio" preload="none"></audio>
<script>
const songs = [
  { title: "Surah 1: Al-Fatiha", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/001.mp3" },
  { title: "Surah 2: Al-Baqarah", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/002.mp3" },
  { title: "Surah 3: Al-Imran", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/003.mp3" },
  { title: "Surah 4: An-Nisa", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/004.mp3" },
  { title: "Surah 5: Al-Maeda", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/005.mp3" }
];

let currentIndex = 0;
let audio = document.getElementById('audio');
let playBtn = document.getElementById('play-btn');

function loadSong(index) {
  let song = songs[index];
  document.getElementById('current-title').textContent = song.title;
  document.getElementById('current-artist').textContent = song.artist;
  audio.src = song.url;
}

function playSong() {
  audio.play();
  playBtn.textContent = '⏸️';
}

function pauseSong() {
  audio.pause();
  playBtn.textContent = '▶️';
}

function togglePlayPause() {
  if (audio.paused) playSong();
  else pauseSong();
}

function nextSong() {
  currentIndex = (currentIndex + 1) % songs.length;
  loadSong(currentIndex);
  playSong();
}

function prevSong() {
  currentIndex = (currentIndex - 1 + songs.length) % songs.length;
  loadSong(currentIndex);
  playSong();
}

function changeVolume(value) {
  audio.volume = value;
}

function seekAudio(value) {
  audio.currentTime = audio.duration * (value / 100);
}

audio.ontimeupdate = () => {
  let current = Math.floor(audio.currentTime);
  let duration = Math.floor(audio.duration);
  document.getElementById('current-time').textContent = formatTime(current);
  document.getElementById('duration').textContent = formatTime(duration);
  document.getElementById('progress-bar').value = (current / duration) * 100;
};

function formatTime(seconds) {
  let min = Math.floor(seconds / 60);
  let sec = Math.floor(seconds % 60);
  return `${min}:${sec < 10 ? '0' : ''}${sec}`;
}

function toggleLoop() {
  audio.loop = !audio.loop;
}

function toggleShuffle() {
  songs.sort(() => Math.random() - 0.5);
  loadSong(0);
}

function renderLibrary() {
  const lib = document.getElementById('library');
  lib.innerHTML = '';
  songs.forEach((song, index) => {
    const item = document.createElement('div');
    item.className = 'song-item';
    item.innerHTML = `<div class="song-info"><span class="song-title">${song.title}</span><small>${song.artist}</small></div>`;
    item.onclick = () => {
      currentIndex = index;
      loadSong(currentIndex);
      playSong();
    };
    lib.appendChild(item);
  });
}

window.onload = () => {
  loadSong(0);
  renderLibrary();
};
</script>
</body>
</html>
