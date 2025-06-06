
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Audio Stream Player</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            overscroll-behavior-y: contain; /* Prevents pull-to-refresh in WebView */
        }
        /* Custom scrollbar for webkit browsers */
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #2d3748; /* bg-gray-700 */
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #4a5568; /* bg-gray-600 */
            border-radius: 3px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
            background: #718096; /* bg-gray-500 */
        }
        /* For Firefox */
        .custom-scrollbar {
            scrollbar-width: thin;
            scrollbar-color: #4a5568 #2d3748;
        }

        .tab-active {
            border-bottom-width: 2px;
            border-color: #3b82f6; /* blue-500 */
            color: #3b82f6;
        }
        .modal {
            display: none; /* Hidden by default */
        }
        .modal.active {
            display: flex; /* Show when active */
        }
        /* Ensure player controls are easily tappable */
        .player-button {
            min-width: 44px;
            min-height: 44px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .song-item {
            border-bottom-width: 1px;
            border-color: #4a5568; /* gray-600 */
        }
        .song-item:last-child {
            border-bottom-width: 0;
        }
        .song-item.selected {
            background-color: #4a5568; /* gray-600 */
        }
         /* Basic Toast Styling */
        .toast {
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background-color: #1f2937; /* bg-gray-800 */
            color: white;
            padding: 10px 20px;
            border-radius: 8px;
            z-index: 1000;
            opacity: 0;
            transition: opacity 0.3s ease-in-out, bottom 0.3s ease-in-out;
            visibility: hidden;
        }

        .toast.show {
            opacity: 1;
            bottom: 30px; /* Slide up */
            visibility: visible;
        }
    </style>
</head>
<body class="bg-gray-900 text-white flex flex-col h-screen">

    <!-- Audio Element -->
    <audio id="audioPlayer"></audio>

    <!-- Top Bar: Current Song Info -->
    <div class="p-4 bg-gray-800 shadow-md">
        <div id="currentSongDisplay" class="text-center">
            <p id="songTitle" class="text-lg font-semibold truncate">No Song Selected</p>
            <p id="songArtist" class="text-sm text-gray-400 truncate">---</p>
        </div>
    </div>

    <!-- Progress Bar and Time -->
    <div class="p-4 bg-gray-800">
        <input type="range" id="progressBar" value="0" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-blue-500">
        <div class="flex justify-between text-xs text-gray-400 mt-1">
            <span id="currentTime">0:00</span>
            <span id="duration">0:00</span>
        </div>
    </div>

    <!-- Player Controls -->
    <div class="p-4 bg-gray-800 flex items-center justify-around">
        <button id="shuffleBtn" class="player-button text-gray-400 hover:text-blue-500"><i class="fas fa-random fa-lg"></i></button>
        <button id="prevBtn" class="player-button text-gray-300 hover:text-white"><i class="fas fa-step-backward fa-xl"></i></button>
        <button id="playPauseBtn" class="player-button text-blue-500 hover:text-blue-400 bg-gray-700 rounded-full w-16 h-16 flex items-center justify-center">
            <i class="fas fa-play fa-2x"></i>
        </button>
        <button id="nextBtn" class="player-button text-gray-300 hover:text-white"><i class="fas fa-step-forward fa-xl"></i></button>
        <button id="loopBtn" class="player-button text-gray-400 hover:text-blue-500"><i class="fas fa-retweet fa-lg"></i></button> <!-- Using retweet as a loop icon -->
    </div>

    <!-- Volume Control -->
    <div class="px-4 pt-2 pb-4 bg-gray-800 flex items-center justify-center space-x-2">
        <i class="fas fa-volume-down text-gray-400"></i>
        <input type="range" id="volumeCtrl" min="0" max="1" step="0.01" value="0.5" class="w-1/2 md:w-1/4 h-1 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-blue-500">
        <i class="fas fa-volume-up text-gray-400"></i>
    </div>

    <!-- Tabs for Song Lists -->
    <div class="flex border-b border-gray-700 sticky top-0 bg-gray-900 z-10">
        <button data-tab="library" class="tab-button flex-1 py-3 text-center text-gray-400 hover:text-white tab-active">Library</button>
        <button data-tab="favorites" class="tab-button flex-1 py-3 text-center text-gray-400 hover:text-white">Favorites</button>
        <button data-tab="playlists" class="tab-button flex-1 py-3 text-center text-gray-400 hover:text-white">Playlists</button>
    </div>

    <!-- Song List Area -->
    <div id="songListContainer" class="flex-grow overflow-y-auto custom-scrollbar p-2">
        <!-- Library View -->
        <div id="libraryView" class="tab-content">
            <!-- Songs will be injected here -->
        </div>
        <!-- Favorites View -->
        <div id="favoritesView" class="tab-content hidden">
            <p class="text-gray-500 text-center p-4 hidden" id="noFavoritesMessage">No favorite songs yet.</p>
            <!-- Favorite songs will be injected here -->
        </div>
        <!-- Playlists View -->
        <div id="playlistsView" class="tab-content hidden">
            <div class="p-2">
                <button id="createPlaylistBtn" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-lg mb-2">
                    <i class="fas fa-plus-circle mr-2"></i>Create New Playlist
                </button>
                 <div id="myPlaylistsContainer">
                    <!-- Playlists will be listed here -->
                 </div>
                 <p class="text-gray-500 text-center p-4 hidden" id="noPlaylistsMessage">No playlists created yet.</p>
            </div>
        </div>
        <!-- Single Playlist Songs View -->
        <div id="singlePlaylistSongsView" class="tab-content hidden">
             <div class="flex items-center justify-between p-2 border-b border-gray-700">
                <button id="backToPlaylistsBtn" class="text-blue-500 hover:text-blue-400">
                    <i class="fas fa-arrow-left mr-2"></i> Back to Playlists
                </button>
                <h3 id="currentPlaylistNameHeader" class="text-lg font-semibold">Playlist Songs</h3>
            </div>
            <div id="songsInPlaylistContainer">
                <!-- Songs in the selected playlist will be injected here -->
            </div>
             <p class="text-gray-500 text-center p-4 hidden" id="noSongsInPlaylistMessage">This playlist is empty.</p>
        </div>
    </div>

    <!-- Modal for Creating Playlist -->
    <div id="createPlaylistModal" class="modal fixed inset-0 bg-black bg-opacity-75 items-center justify-center z-50 p-4">
        <div class="bg-gray-800 p-6 rounded-lg shadow-xl w-full max-w-md">
            <h3 class="text-xl font-semibold mb-4">Create New Playlist</h3>
            <input type="text" id="newPlaylistName" placeholder="Playlist Name" class="w-full p-2 rounded bg-gray-700 border border-gray-600 focus:border-blue-500 outline-none mb-4">
            <div class="flex justify-end space-x-2">
                <button id="cancelCreatePlaylistBtn" class="bg-gray-600 hover:bg-gray-700 text-white font-semibold py-2 px-4 rounded-lg">Cancel</button>
                <button id="savePlaylistBtn" class="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-lg">Create</button>
            </div>
        </div>
    </div>

    <!-- Modal for Adding to Playlist -->
    <div id="addToPlaylistModal" class="modal fixed inset-0 bg-black bg-opacity-75 items-center justify-center z-50 p-4">
        <div class="bg-gray-800 p-6 rounded-lg shadow-xl w-full max-w-md">
            <h3 class="text-xl font-semibold mb-4">Add to Playlist</h3>
            <input type="hidden" id="songIdToAddToPlaylist">
            <div id="playlistSelectionContainer" class="max-h-60 overflow-y-auto custom-scrollbar mb-4 border border-gray-700 rounded-md p-2">
                <!-- Available playlists for selection -->
            </div>
            <p id="noPlaylistsForAddingMessage" class="text-gray-400 mb-4 hidden">No playlists available. Create one first!</p>
            <div class="flex justify-end space-x-2">
                <button id="cancelAddToPlaylistBtn" class="bg-gray-600 hover:bg-gray-700 text-white font-semibold py-2 px-4 rounded-lg">Cancel</button>
                <!-- Save button will be dynamically handled or one per playlist item -->
            </div>
        </div>
    </div>
    
    <!-- Toast Notification -->
    <div id="toastNotification" class="toast">This is a toast message!</div>


    <script type="module">
        // Firebase Imports
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, addDoc, getDocs, arrayUnion, arrayRemove } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js"; // Added arrayUnion, arrayRemove

        // --- Firebase Configuration ---
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
            // Fallback config if __firebase_config is not injected (for local testing)
            // PASTE YOUR ACTUAL FIREBASE CONFIG HERE
            apiKey: "AIzaSyDtGUyB9bpiL9JefwLnP0AEHg9G9QzFQAE",
  authDomain: "audio-stream-player-4ed4d.firebaseapp.com",
  projectId: "audio-stream-player-4ed4d",
  storageBucket: "audio-stream-player-4ed4d.firebasestorage.app",
  messagingSenderId: "498762969730",
  appId: "1:498762969730:web:6474cbafe2ed2c8ef9ba9b",
  measurementId: "G-SR6D3J302T"
        };
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'audio-player-default-app';

        // Initialize Firebase
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        // import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js"; // For debugging
        // setLogLevel('debug'); // Uncomment for Firestore debugging

        let userId = null;
        let dbUserDocRef = null; // Changed from dbFavoritesRef, this will be the main user document
        let dbPlaylistsCollectionRef = null;
        let unsubscribeUserDoc = null; // Changed from unsubscribeFavorites
        let unsubscribePlaylists = null;


        // --- Audio Player State & Elements ---
        const audioPlayer = document.getElementById('audioPlayer');
        const playPauseBtn = document.getElementById('playPauseBtn');
        const prevBtn = document.getElementById('prevBtn');
        const nextBtn = document.getElementById('nextBtn');
        const loopBtn = document.getElementById('loopBtn');
        const shuffleBtn = document.getElementById('shuffleBtn');
        const progressBar = document.getElementById('progressBar');
        const currentTimeDisplay = document.getElementById('currentTime');
        const durationDisplay = document.getElementById('duration');
        const volumeCtrl = document.getElementById('volumeCtrl');
        const songTitleDisplay = document.getElementById('songTitle');
        const songArtistDisplay = document.getElementById('songArtist');

        const libraryView = document.getElementById('libraryView');
        const favoritesView = document.getElementById('favoritesView');
        const playlistsView = document.getElementById('playlistsView');
        const singlePlaylistSongsView = document.getElementById('singlePlaylistSongsView');
        const songsInPlaylistContainer = document.getElementById('songsInPlaylistContainer');
        const currentPlaylistNameHeader = document.getElementById('currentPlaylistNameHeader');
        const backToPlaylistsBtn = document.getElementById('backToPlaylistsBtn');
        
        const noFavoritesMessage = document.getElementById('noFavoritesMessage');
        const noPlaylistsMessage = document.getElementById('noPlaylistsMessage');
        const myPlaylistsContainer = document.getElementById('myPlaylistsContainer');
        const noSongsInPlaylistMessage = document.getElementById('noSongsInPlaylistMessage');


        // Modals
        const createPlaylistModal = document.getElementById('createPlaylistModal');
        const newPlaylistNameInput = document.getElementById('newPlaylistName');
        const savePlaylistBtn = document.getElementById('savePlaylistBtn');
        const cancelCreatePlaylistBtn = document.getElementById('cancelCreatePlaylistBtn');
        const createPlaylistBtn = document.getElementById('createPlaylistBtn');

        const addToPlaylistModal = document.getElementById('addToPlaylistModal');
        const playlistSelectionContainer = document.getElementById('playlistSelectionContainer');
        const songIdToAddToPlaylistInput = document.getElementById('songIdToAddToPlaylist');
        const cancelAddToPlaylistBtn = document.getElementById('cancelAddToPlaylistBtn');
        const noPlaylistsForAddingMessage = document.getElementById('noPlaylistsForAddingMessage');
        
        const toastNotification = document.getElementById('toastNotification');


        let songs = [
            { id: 's1', title: 'Surah 1: Al-Fatiha', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/001.mp3'},
            { id: 's2', title: 'Surah 2: Al-Baqarah', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/002.mp3'},
            { id: 's3', title: 'Surah 3: Al-Imran', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/003.mp3'},
            { id: 's4', title: 'Surah 4: An-Nisa', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/004.mp3'},
            { id: 's5', title: 'Surah 5: Al-Maeda', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/005.mp3'},
            { id: 's6', title: 'Surah 6: Al-Annam', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/006.mp3'},
            { id: 's7', title: 'Surah 7: Al-Araf', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/007.mp3'},
            { id: 's8', title: 'Surah 8: Al-Anfal', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/008.mp3'},
            { id: 's9', title: 'Surah 9: At-Tawba', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/009.mp3'},
            { id: 's10', title: 'Surah 10: Yunus', artist: 'Mishary Al-Afasy and Ibrahim Walk', url: 'https://archive.org/download/AlQuranWithEnglishSaheehIntlTranslation--RecitationByMishariIbnRashidAl-AfasyWithIbrahimWalk/010.mp3'}
        ];

        let currentSongIndex = 0;
        let isPlaying = false;
        let isShuffle = false;
        let isLoop = false;
        let currentTracklist = [...songs]; 
        let originalOrderTracklist = [...songs]; 
        let favoriteSongIds = []; // Renamed from 'favorites'
        let playlists = []; 
        let currentOpenPlaylistId = null;

        // --- Toast Notification ---
        function showToast(message, duration = 3000) {
            toastNotification.textContent = message;
            toastNotification.classList.add('show');
            setTimeout(() => {
                toastNotification.classList.remove('show');
            }, duration);
        }

        // --- Authentication and Data Loading ---
        onAuthStateChanged(auth, async (user) => {
            if (user) {
                userId = user.uid;
                console.log("User authenticated with UID:", userId);
                // Path to the user's specific document: artifacts/{appId}/users/{userId}
                dbUserDocRef = doc(db, "artifacts", appId, "users", userId);
                // Path to the playlists subcollection for this user: artifacts/{appId}/users/{userId}/playlists
                dbPlaylistsCollectionRef = collection(dbUserDocRef, "playlists");
                
                loadUserDocumentData(); // Loads favorites and potentially other user-specific settings
                loadPlaylists();
            } else {
                userId = null;
                console.log("User not authenticated. Player data will not be saved.");
                if (unsubscribeUserDoc) unsubscribeUserDoc();
                if (unsubscribePlaylists) unsubscribePlaylists();
                favoriteSongIds = []; // Reset local data
                playlists = [];
                renderFavorites(); // Update UI
                renderPlaylists();
            }
            // Initial UI setup after auth state is known (or not known)
            loadSong(currentSongIndex); 
            renderSongList(libraryView, songs); 
            updateActiveTab('library');
        });

        async function initAuth() {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (error) {
                console.error("Authentication error:", error);
                showToast("Error connecting to services. Data might not save.", 5000);
                if (!userId) { // If auth fails and user still not set
                    userId = 'tempUser_' + Date.now(); // Fallback, but data won't persist correctly
                    console.warn("Using temporary non-persistent user ID due to auth failure:", userId);
                    // Setup refs with temp ID, mostly for local UI testing without real persistence
                    dbUserDocRef = doc(db, "artifacts", appId, "users", userId);
                    dbPlaylistsCollectionRef = collection(dbUserDocRef, "playlists");
                    // Attempt to load UI elements even if data won't save
                    loadSong(currentSongIndex);
                    renderSongList(libraryView, songs);
                    updateActiveTab('library');
                }
            }
        }


        // --- Audio Controls ---
        function loadSong(index) {
            if (index >= 0 && index < currentTracklist.length) {
                const song = currentTracklist[index];
                audioPlayer.src = song.url;
                songTitleDisplay.textContent = song.title;
                songArtistDisplay.textContent = song.artist;
                currentSongIndex = index; 
                updateSelectedSongUI();
                if (isPlaying) audioPlayer.play().catch(e => console.error("Error playing loaded song:", e));
            } else {
                console.warn("Invalid song index:", index, "Tracklist length:", currentTracklist.length);
                if (isLoop && currentTracklist.length > 0) { 
                    currentSongIndex = 0;
                    loadSong(currentSongIndex);
                    if (isPlaying) playSong();
                } else {
                    pauseSong(); 
                    currentSongIndex = 0; 
                    if (currentTracklist.length > 0) loadSong(currentSongIndex); 
                    else { // No songs in tracklist
                        songTitleDisplay.textContent = "No Songs in Playlist";
                        songArtistDisplay.textContent = "---";
                    }
                }
            }
        }

        function playSong() {
            if (!audioPlayer.src && currentTracklist.length > 0) {
                loadSong(currentSongIndex); // Ensure song is loaded if src is missing
            }
            if (audioPlayer.src) { // Only play if there's a source
                audioPlayer.play()
                    .then(() => {
                        isPlaying = true;
                        playPauseBtn.innerHTML = '<i class="fas fa-pause fa-2x"></i>';
                    })
                    .catch(error => {
                        console.error("Error playing audio:", error);
                        showToast("Error playing audio. Check URL or network.", 4000);
                        isPlaying = false;
                        playPauseBtn.innerHTML = '<i class="fas fa-play fa-2x"></i>';
                    });
            } else {
                showToast("No song to play.", 3000);
            }
        }

        function pauseSong() {
            audioPlayer.pause();
            isPlaying = false;
            playPauseBtn.innerHTML = '<i class="fas fa-play fa-2x"></i>';
        }

        playPauseBtn.addEventListener('click', () => {
            if (isPlaying) {
                pauseSong();
            } else {
                playSong();
            }
        });

        nextBtn.addEventListener('click', () => {
            let nextIndex = currentSongIndex + 1;
            if (nextIndex >= currentTracklist.length) {
                if (currentTracklist.length > 0) nextIndex = 0; 
                else return; // No songs to play
            }
            loadSong(nextIndex);
        });

        prevBtn.addEventListener('click', () => {
            let prevIndex = currentSongIndex - 1;
            if (prevIndex < 0) {
                if (currentTracklist.length > 0) prevIndex = currentTracklist.length - 1;
                else return; // No songs to play
            }
            loadSong(prevIndex);
        });

        loopBtn.addEventListener('click', () => {
            isLoop = !isLoop;
            // HTML5 audio 'loop' attribute is for single track. 
            // Playlist loop is handled by 'ended' event and nextBtn logic.
            audioPlayer.loop = isLoop; 
            loopBtn.classList.toggle('text-blue-500', isLoop);
            loopBtn.classList.toggle('text-gray-400', !isLoop);
            showToast(isLoop ? "Loop current track enabled" : "Loop current track disabled");
        });

        shuffleBtn.addEventListener('click', () => {
            isShuffle = !isShuffle;
            shuffleBtn.classList.toggle('text-blue-500', isShuffle);
            shuffleBtn.classList.toggle('text-gray-400', !isShuffle);
            
            const currentPlayingSongId = currentTracklist[currentSongIndex]?.id;

            if (isShuffle) {
                originalOrderTracklist = [...currentTracklist]; // Save the order before shuffle
                currentTracklist.sort(() => Math.random() - 0.5);
                showToast("Shuffle enabled");
            } else {
                currentTracklist = [...originalOrderTracklist]; // Revert to original order
                showToast("Shuffle disabled");
            }
            
            // Find the new index of the currently playing song
            const newIdx = currentTracklist.findIndex(s => s.id === currentPlayingSongId);
            currentSongIndex = (newIdx !== -1) ? newIdx : 0; // Fallback to first song if not found
            
            const activeTab = document.querySelector('.tab-button.tab-active').dataset.tab;
            if (activeTab === 'library') renderSongList(libraryView, currentTracklist);
            else if (activeTab === 'favorites') renderFavorites(); // This will re-filter and re-render
            else if (activeTab === 'playlists' && currentOpenPlaylistId) renderSongsInPlaylist(currentOpenPlaylistId); // This will re-filter and re-render
            
            // No need to call loadSong() here unless we want to immediately change the displayed info
            // The actual audio source doesn't change until next/prev or new song click
            updateSelectedSongUI(); 
        });
        

        audioPlayer.addEventListener('timeupdate', () => {
            if (audioPlayer.duration) {
                const progress = (audioPlayer.currentTime / audioPlayer.duration) * 100;
                progressBar.value = progress;
                currentTimeDisplay.textContent = formatTime(audioPlayer.currentTime);
            }
        });

        audioPlayer.addEventListener('loadedmetadata', () => {
            durationDisplay.textContent = formatTime(audioPlayer.duration);
            progressBar.value = 0; 
        });
        
        audioPlayer.addEventListener('ended', () => {
            if (!audioPlayer.loop) { // If single track loop is OFF
                nextBtn.click();
            }
        });


        progressBar.addEventListener('input', () => {
            if (audioPlayer.duration) {
                const seekTime = (progressBar.value / 100) * audioPlayer.duration;
                audioPlayer.currentTime = seekTime;
            }
        });

        volumeCtrl.addEventListener('input', (e) => {
            audioPlayer.volume = e.target.value;
        });

        function formatTime(seconds) {
            const minutes = Math.floor(seconds / 60);
            const secs = Math.floor(seconds % 60);
            return `${minutes}:${secs < 10 ? '0' : ''}${secs}`;
        }

        // --- Song List Rendering ---
        function renderSongList(container, tracklistToRender, isPlaylistContext = false, playlistIdForContext = null) {
            container.innerHTML = ''; 
            if (!tracklistToRender || tracklistToRender.length === 0) {
                 if (container === libraryView) container.innerHTML = '<p class="text-gray-500 text-center p-4">Library is empty.</p>';
                return;
            }

            tracklistToRender.forEach((song) => { // Removed index, not directly used here
                const songDiv = document.createElement('div');
                songDiv.className = 'song-item p-3 flex justify-between items-center cursor-pointer hover:bg-gray-700 transition-colors duration-150';
                
                // Check if this song from tracklistToRender is the currently playing song in the global currentTracklist
                if (song.id === currentTracklist[currentSongIndex]?.id ) {
                     songDiv.classList.add('selected', 'border-l-4', 'border-blue-500');
                }
                songDiv.dataset.songId = song.id;

                songDiv.innerHTML = `
                    <div class="flex-grow" data-action="play">
                        <p class="font-semibold truncate">${song.title}</p>
                        <p class="text-sm text-gray-400 truncate">${song.artist}</p>
                    </div>
                    <div class="flex items-center space-x-3 ml-2">
                        <button data-action="toggleFavorite" data-song-id="${song.id}" class="text-gray-400 hover:text-pink-500 p-2 rounded-full focus:outline-none">
                            <i class="fas fa-heart ${favoriteSongIds.includes(song.id) ? 'text-pink-500' : ''}"></i>
                        </button>
                        ${isPlaylistContext ? 
                            `<button data-action="removeFromPlaylist" data-song-id="${song.id}" data-playlist-id="${playlistIdForContext}" class="text-gray-400 hover:text-red-500 p-2 rounded-full focus:outline-none">
                                <i class="fas fa-trash-alt"></i>
                            </button>` :
                            `<button data-action="addToPlaylist" data-song-id="${song.id}" class="text-gray-400 hover:text-blue-500 p-2 rounded-full focus:outline-none">
                                <i class="fas fa-plus-circle"></i>
                            </button>`
                        }
                    </div>
                `;
                
                songDiv.addEventListener('click', (e) => {
                    const actionTarget = e.target.closest('button') || e.target.closest('[data-action="play"]');
                    if (!actionTarget) return;

                    const action = actionTarget.dataset.action;
                    const clickedSongId = songDiv.dataset.songId;

                    if (action === 'play') {
                        // Update currentTracklist to reflect the list this song was clicked from
                        currentTracklist = [...tracklistToRender]; 
                        originalOrderTracklist = [...tracklistToRender]; // For unshuffle
                        if(isShuffle) { 
                             currentTracklist = [...originalOrderTracklist].sort(() => Math.random() - 0.5);
                        }
                        const songIndexInClickedList = currentTracklist.findIndex(s => s.id === clickedSongId);

                        if (songIndexInClickedList !== -1) {
                            currentSongIndex = songIndexInClickedList;
                            loadSong(currentSongIndex);
                            playSong();
                        }
                    } else if (action === 'toggleFavorite') {
                        toggleFavorite(clickedSongId);
                    } else if (action === 'addToPlaylist') {
                        openAddToPlaylistModal(clickedSongId);
                    } else if (action === 'removeFromPlaylist') {
                        const playlistId = actionTarget.dataset.playlistId;
                        removeSongFromPlaylist(clickedSongId, playlistId);
                    }
                    updateSelectedSongUI(); 
                });
                container.appendChild(songDiv);
            });
        }
        
        function updateSelectedSongUI() {
            document.querySelectorAll('.song-item').forEach(item => {
                item.classList.remove('selected', 'border-l-4', 'border-blue-500');
                const songId = item.dataset.songId;
                
                // Highlight if the item's song ID matches the current song in the global currentTracklist
                if (songId === currentTracklist[currentSongIndex]?.id) {
                    const parentView = item.closest('.tab-content');
                    const activeTab = document.querySelector('.tab-button.tab-active').dataset.tab;
                    let isActiveViewItem = false;

                    if (activeTab === 'library' && parentView?.id === 'libraryView') isActiveViewItem = true;
                    if (activeTab === 'favorites' && parentView?.id === 'favoritesView') isActiveViewItem = true;
                    if (activeTab === 'playlists') { 
                        if (currentOpenPlaylistId && parentView?.id === 'singlePlaylistSongsView') isActiveViewItem = true;
                    }

                    if(isActiveViewItem){
                        item.classList.add('selected', 'border-l-4', 'border-blue-500');
                        // item.scrollIntoView({ behavior: 'smooth', block: 'nearest' }); // Can be jerky, consider enabling if really needed
                    }
                }
            });
        }


        // --- Tab Navigation ---
        const tabButtons = document.querySelectorAll('.tab-button');
        const tabContents = document.querySelectorAll('.tab-content');

        tabButtons.forEach(button => {
            button.addEventListener('click', () => {
                const tabName = button.dataset.tab;
                updateActiveTab(tabName);
            });
        });
        
        function updateActiveTab(tabName) {
            tabButtons.forEach(btn => {
                btn.classList.toggle('tab-active', btn.dataset.tab === tabName);
                btn.classList.toggle('text-gray-400', btn.dataset.tab !== tabName);
            });
            tabContents.forEach(content => {
                content.classList.add('hidden');
            });

            currentOpenPlaylistId = null; 

            if (tabName === 'library') {
                document.getElementById('libraryView').classList.remove('hidden');
                currentTracklist = isShuffle ? [...songs].sort(() => Math.random() - 0.5) : [...songs];
                originalOrderTracklist = [...songs];
                renderSongList(libraryView, currentTracklist);
            } else if (tabName === 'favorites') {
                document.getElementById('favoritesView').classList.remove('hidden');
                renderFavorites(); // This will also set currentTracklist if favorites are shown
            } else if (tabName === 'playlists') {
                document.getElementById('playlistsView').classList.remove('hidden');
                singlePlaylistSongsView.classList.add('hidden'); 
                renderPlaylists();
            }
            updateSelectedSongUI();
        }


        // --- Favorites Management (Uses dbUserDocRef) ---
        async function loadUserDocumentData() {
            if (!userId || !dbUserDocRef) {
                 favoriteSongIds = []; renderFavorites(); return;
            }
            if (unsubscribeUserDoc) unsubscribeUserDoc(); 

            unsubscribeUserDoc = onSnapshot(dbUserDocRef, (docSnap) => {
                if (docSnap.exists()) {
                    const data = docSnap.data();
                    favoriteSongIds = data.favoriteSongIds || [];
                } else {
                    favoriteSongIds = [];
                    // User document doesn't exist yet, it will be created on first favorite.
                    console.log("User document does not exist yet for UID:", userId);
                }
                renderFavorites(); // Update the favorites view
                // Re-render any visible song list to update heart icons
                const activeTabButton = document.querySelector('.tab-button.tab-active');
                if (!activeTabButton) return;
                const activeTab = activeTabButton.dataset.tab;

                if (activeTab === 'library') renderSongList(libraryView, currentTracklist);
                else if (activeTab === 'playlists' && currentOpenPlaylistId) renderSongsInPlaylist(currentOpenPlaylistId);
                updateSelectedSongUI();

            }, (error) => {
                console.error("Error loading user document (favorites):", error);
                showToast("Could not load favorites.", 4000);
                favoriteSongIds = []; 
                renderFavorites();
            });
        }

        async function toggleFavorite(songId) {
            if (!userId || !dbUserDocRef) {
                showToast("Cannot save favorite. Not connected.", 3000); return;
            }
            
            const isCurrentlyFavorite = favoriteSongIds.includes(songId);
            const operation = isCurrentlyFavorite ? arrayRemove(songId) : arrayUnion(songId);
            const newFavoriteStatus = !isCurrentlyFavorite;

            try {
                // Use setDoc with merge: true to create the document if it doesn't exist,
                // or update it if it does.
                await setDoc(dbUserDocRef, { 
                    favoriteSongIds: operation 
                }, { merge: true });

                showToast(newFavoriteStatus ? "Added to Favorites" : "Removed from Favorites");
                // onSnapshot (unsubscribeUserDoc) will trigger UI updates for favorites list and icons.
            } catch (error) {
                console.error("Error updating favorites:", error);
                showToast("Error saving favorite.", 3000);
            }
        }

        function renderFavorites() {
            const favSongsData = songs.filter(song => favoriteSongIds.includes(song.id));
            noFavoritesMessage.classList.toggle('hidden', favSongsData.length > 0);
            renderSongList(favoritesView, favSongsData);
            // If favorites tab is active, set its songs as the current tracklist
             if (document.getElementById('favoritesView').classList.contains('hidden') === false) {
                currentTracklist = isShuffle ? [...favSongsData].sort(() => Math.random() - 0.5) : [...favSongsData];
                originalOrderTracklist = [...favSongsData]; // For unshuffle
                if (favSongsData.length === 0) { // If favorites become empty while viewing
                    songTitleDisplay.textContent = "No Songs in Favorites";
                    songArtistDisplay.textContent = "---";
                    pauseSong();
                } else if (!favSongsData.find(s => s.id === currentTracklist[currentSongIndex]?.id)){
                    // If current playing song was removed from favorites, load first fav song
                    currentSongIndex = 0;
                    loadSong(currentSongIndex);
                }
            }
            updateSelectedSongUI();
        }

        // --- Playlist Management (Uses dbPlaylistsCollectionRef) ---
        createPlaylistBtn.addEventListener('click', () => {
            newPlaylistNameInput.value = '';
            createPlaylistModal.classList.add('active');
            newPlaylistNameInput.focus();
        });
        cancelCreatePlaylistBtn.addEventListener('click', () => createPlaylistModal.classList.remove('active'));

        savePlaylistBtn.addEventListener('click', async () => {
            if (!userId || !dbPlaylistsCollectionRef) {
                 showToast("Cannot save playlist. Not connected.", 3000); return;
            }
            const playlistName = newPlaylistNameInput.value.trim();
            if (!playlistName) {
                showToast("Playlist name cannot be empty.", 3000); return;
            }
            if (playlists.find(p => p.name.toLowerCase() === playlistName.toLowerCase())) {
                showToast("A playlist with this name already exists.", 3000); return;
            }
            try {
                await addDoc(dbPlaylistsCollectionRef, {
                    name: playlistName,
                    songIds: [], // Initialize with an empty array of song IDs
                    createdAt: new Date() // Optional: timestamp for sorting
                });
                createPlaylistModal.classList.remove('active');
                showToast(`Playlist "${playlistName}" created.`);
                // onSnapshot (unsubscribePlaylists) will handle UI update
            } catch (error) {
                console.error("Error creating playlist:", error);
                showToast("Error creating playlist.", 3000);
            }
        });

        async function loadPlaylists() {
            if (!userId || !dbPlaylistsCollectionRef) {
                 playlists = []; renderPlaylists(); return;
            }
            if (unsubscribePlaylists) unsubscribePlaylists();

            // Query playlists, optionally order by name or creation date if you add one
            const q = query(dbPlaylistsCollectionRef /*, orderBy("name") // Requires composite index */);
            unsubscribePlaylists = onSnapshot(q, (querySnapshot) => {
                playlists = [];
                querySnapshot.forEach((doc) => {
                    playlists.push({ id: doc.id, ...doc.data() });
                });
                // Sort playlists by name client-side if not ordered by Firestore query
                playlists.sort((a,b) => a.name.localeCompare(b.name));

                renderPlaylists();
                if (currentOpenPlaylistId) {
                    const stillExists = playlists.find(p => p.id === currentOpenPlaylistId);
                    if (stillExists) {
                        renderSongsInPlaylist(currentOpenPlaylistId); // Refresh songs if playlist data changed
                    } else { 
                        showPlaylistsList(); 
                        currentOpenPlaylistId = null;
                    }
                }
            }, (error) => {
                console.error("Error loading playlists:", error);
                showToast("Could not load playlists.", 4000);
                playlists = []; 
                renderPlaylists();
            });
        }
        
        function renderPlaylists() {
            myPlaylistsContainer.innerHTML = '';
            noPlaylistsMessage.classList.toggle('hidden', playlists.length > 0);

            if (playlists.length > 0) {
                playlists.forEach(playlist => {
                    const div = document.createElement('div');
                    div.className = 'p-3 bg-gray-800 rounded-lg mb-2 flex justify-between items-center cursor-pointer hover:bg-gray-700 transition-colors duration-150';
                    div.innerHTML = `
                        <span class="font-semibold truncate flex-grow">${playlist.name} (${playlist.songIds?.length || 0} songs)</span>
                        <div class="flex-shrink-0">
                            <button data-playlist-id="${playlist.id}" data-action="delete-playlist" class="text-red-500 hover:text-red-400 p-2 rounded-full focus:outline-none"><i class="fas fa-trash-alt"></i></button>
                        </div>
                    `;
                    div.addEventListener('click', (e) => {
                        const buttonTarget = e.target.closest('button[data-action="delete-playlist"]');
                        if (buttonTarget) {
                            const playlistId = buttonTarget.dataset.playlistId;
                            // Custom modal for confirm would be better
                            if (confirm(`Are you sure you want to delete the playlist "${playlist.name}"? This cannot be undone.`)) {
                                deletePlaylist(playlistId);
                            }
                        } else {
                            showSongsForPlaylist(playlist.id);
                        }
                    });
                    myPlaylistsContainer.appendChild(div);
                });
            }
        }

        async function deletePlaylist(playlistId) {
             if (!userId || !dbPlaylistsCollectionRef) {
                 showToast("Cannot delete playlist. Not connected.", 3000); return;
            }
            try {
                await deleteDoc(doc(dbPlaylistsCollectionRef, playlistId));
                showToast("Playlist deleted.");
                // onSnapshot will update the UI. If this playlist was open, it will be handled.
            } catch (error) {
                console.error("Error deleting playlist:", error);
                showToast("Error deleting playlist.", 3000);
            }
        }

        function openAddToPlaylistModal(songId) {
            if (!userId || !dbPlaylistsCollectionRef) { showToast("Connect to save changes.", 3000); return; }
            songIdToAddToPlaylistInput.value = songId;
            playlistSelectionContainer.innerHTML = '';
            noPlaylistsForAddingMessage.classList.toggle('hidden', playlists.length > 0);

            if (playlists.length > 0) {
                playlists.forEach(playlist => {
                    const isSongInPlaylist = playlist.songIds?.includes(songId);

                    const pDiv = document.createElement('div');
                    pDiv.className = 'p-2 border-b border-gray-700 flex justify-between items-center last:border-b-0';
                    pDiv.innerHTML = `
                        <span class="truncate">${playlist.name}</span>
                        <button data-playlist-id="${playlist.id}" ${isSongInPlaylist ? 'disabled class="text-gray-500 cursor-not-allowed p-1 rounded"' : 'class="text-blue-500 hover:text-blue-400 p-1 rounded"'}">
                            ${isSongInPlaylist ? '<i class="fas fa-check-circle mr-1"></i> Added' : '<i class="fas fa-plus-circle mr-1"></i> Add'}
                        </button>
                    `;
                    if (!isSongInPlaylist) {
                        pDiv.querySelector('button').addEventListener('click', async () => {
                            await addSongToPlaylist(songId, playlist.id);
                            addToPlaylistModal.classList.remove('active'); 
                        });
                    }
                    playlistSelectionContainer.appendChild(pDiv);
                });
            }
            addToPlaylistModal.classList.add('active');
        }
        cancelAddToPlaylistBtn.addEventListener('click', () => addToPlaylistModal.classList.remove('active'));

        async function addSongToPlaylist(songId, playlistId) {
            if (!userId || !dbPlaylistsCollectionRef) {
                 showToast("Cannot add to playlist. Not connected.", 3000); return;
            }
            const playlistRef = doc(dbPlaylistsCollectionRef, playlistId);
            try {
                // Use arrayUnion to add the songId if it's not already present
                await updateDoc(playlistRef, {
                    songIds: arrayUnion(songId)
                });
                const playlist = playlists.find(p => p.id === playlistId);
                showToast(`Added to playlist "${playlist?.name || 'playlist'}".`);
                // onSnapshot will handle UI refresh
            } catch (error) {
                console.error("Error adding song to playlist:", error);
                showToast("Error adding song to playlist.", 3000);
            }
        }

        async function removeSongFromPlaylist(songId, playlistId) {
             if (!userId || !dbPlaylistsCollectionRef) {
                 showToast("Cannot remove from playlist. Not connected.", 3000); return;
            }
            const playlistRef = doc(dbPlaylistsCollectionRef, playlistId);
            try {
                // Use arrayRemove to remove the songId
                await updateDoc(playlistRef, {
                    songIds: arrayRemove(songId)
                });
                const playlist = playlists.find(p => p.id === playlistId);
                showToast(`Removed from playlist "${playlist?.name || 'playlist'}".`);
                // onSnapshot handles UI update.
            } catch (error) {
                console.error("Error removing song from playlist:", error);
                showToast("Error removing song.", 3000);
            }
        }

        function showSongsForPlaylist(playlistId) {
            const playlist = playlists.find(p => p.id === playlistId);
            if (!playlist) {
                console.error("Playlist not found:", playlistId);
                showPlaylistsList(); 
                return;
            }
            currentOpenPlaylistId = playlistId; // Set this before rendering

            document.getElementById('playlistsView').classList.add('hidden');
            singlePlaylistSongsView.classList.remove('hidden');
            currentPlaylistNameHeader.textContent = playlist.name;
            renderSongsInPlaylist(playlistId); // This will now correctly set currentTracklist
        }
        
        function renderSongsInPlaylist(playlistId) {
            const playlist = playlists.find(p => p.id === playlistId);
            songsInPlaylistContainer.innerHTML = ''; // Clear previous

            if (!playlist || !playlist.songIds || playlist.songIds.length === 0) { 
                noSongsInPlaylistMessage.classList.remove('hidden');
                currentTracklist = []; // Clear tracklist if playlist is empty
                originalOrderTracklist = [];
                loadSong(0); // Attempt to load "no song" state
                return;
            }
            noSongsInPlaylistMessage.classList.add('hidden');
            
            const playlistSongsData = songs.filter(song => playlist.songIds.includes(song.id));
            renderSongList(songsInPlaylistContainer, playlistSongsData, true, playlistId);

            // If this playlist view is active, set its songs as the current tracklist
            if (singlePlaylistSongsView.classList.contains('hidden') === false) {
                currentTracklist = isShuffle ? [...playlistSongsData].sort(() => Math.random() - 0.5) : [...playlistSongsData];
                originalOrderTracklist = [...playlistSongsData]; 

                // If current song is not in this new tracklist, or tracklist became empty
                if (!currentTracklist.find(s => s.id === currentTracklist[currentSongIndex]?.id)) {
                    currentSongIndex = 0; // Reset to first song of this playlist
                    loadSong(currentSongIndex); // Load it (will handle empty tracklist inside)
                } else {
                    // If current song IS in new tracklist, ensure index is correct for this tracklist
                    currentSongIndex = currentTracklist.findIndex(s => s.id === currentTracklist[currentSongIndex]?.id);
                }
            }
            updateSelectedSongUI();
        }


        backToPlaylistsBtn.addEventListener('click', showPlaylistsList);

        function showPlaylistsList() {
            singlePlaylistSongsView.classList.add('hidden');
            document.getElementById('playlistsView').classList.remove('hidden');
            currentOpenPlaylistId = null; // Clear the open playlist context
            // Revert tracklist to library view when going back to the list of playlists
            // This makes navigation more predictable.
            const librarySongs = isShuffle ? [...songs].sort(() => Math.random() - 0.5) : [...songs];
            if (JSON.stringify(currentTracklist) !== JSON.stringify(librarySongs)) {
                 currentTracklist = librarySongs;
                 originalOrderTracklist = [...songs];
                 // currentSongIndex should ideally point to the song that was playing from library, or default
                 // For simplicity, let's find current displayed song or default to 0
                 const playingSongId = audioPlayer.src ? songs.find(s => s.url === audioPlayer.src)?.id : null;
                 let newIndex = 0;
                 if(playingSongId) {
                    const idxInLibrary = currentTracklist.findIndex(s => s.id === playingSongId);
                    if(idxInLibrary !== -1) newIndex = idxInLibrary;
                 }
                 currentSongIndex = newIndex;
                 loadSong(currentSongIndex); // Visually update displayed song if tracklist changed
            }
            updateSelectedSongUI(); 
        }


        // --- Initial Setup ---
        document.addEventListener('DOMContentLoaded', () => {
            initAuth(); 
        });

    </script>
</body>
</html>

