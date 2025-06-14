<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Audio Stream Player</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f2f2f2;
    color: #333;
  }
  .player {
    position: fixed;
    bottom: 0;
    width: 100%;
    background: #fff;
    padding: 10px;
    box-shadow: 0 -2px 10px rgba(0,0,0,0.1);
    z-index: 999;
  }
  .controls, .progress, .info {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
  }
  .controls button {
    margin: 5px;
  }
  .library {
    padding-bottom: 200px;
    overflow-y: auto;
    padding: 20px;
  }
  .song-item {
    margin: 5px 0;
    padding: 10px;
    background: #fff;
    border-radius: 5px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  }
  .favorites, .playlists {
    margin-top: 20px;
  }
</style>
</head>
<body>

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
    <input type="range" min="0" max="1" step="0.01" onchange="changeVolume(this.value)">
  </div>
  <div class="progress">
    <span id="current-time">0:00</span>
    <input type="range" id="progress-bar" value="0" onchange="seekAudio(this.value)">
    <span id="duration">0:00</span>
  </div>
</div>

<audio id="audio" preload="none"></audio>

<script>
const songs = [
  { title: "Surah 1: Al-Fatiha", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/001.mp3" },
  { title: "Surah 2: Al-Baqarah", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/002.mp3" },
  { title: "Surah 3: Al-Imran", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/003.mp3" },
  { title: "Surah 4: An-Nisa", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/004.mp3" },
  { title: "Surah 5: Al-Maeda", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/005.mp3" },
  { title: "Surah 6: Al-Annam", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/006.mp3" },
  { title: "Surah 7: Al-Araf", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/007.mp3" },
  { title: "Surah 8: Al-Anfal", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/008.mp3" },
  { title: "Surah 9: At-Tawba", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/009.mp3" },
  { title: "Surah 10: Yunus", artist: "Mishary Al-Afasy and Ibrahim Walk", url: "https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/010.mp3" }
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
    item.textContent = `${song.title} — ${song.artist}`;
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
