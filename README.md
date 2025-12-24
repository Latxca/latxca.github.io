<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="google-site-verification" content="FNeUzJ5ovoIEIArxoQESilUn_ql1SL1yriR1Dvo2Z_4" />
    <title>Trist's YouTube Client</title>
    <!-- Tailwind CSS CDN for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for the app */
        body {
            font-family: "Inter", sans-serif;
            background-color: #f3f4f6;
        }
        .video-thumbnail {
            cursor: pointer;
            transition: transform 0.2s ease-in-out;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            height: 100%;
        }
        .video-thumbnail:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
        }
        .iframe-container {
            position: relative;
            width: 100%;
            padding-bottom: 56.25%;
            height: 0;
            overflow: hidden;
            border-radius: 0.5rem;
            background: #000;
        }
        .iframe-container iframe {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            border-radius: 0.5rem;
        }
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            border-left-color: #3b82f6;
            border-radius: 50%;
            width: 20px;
            height: 20px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
        .music-warning {
            font-size: 0.75rem;
            color: #ef4444;
            background-color: #fee2e2;
            padding: 0.25rem 0.5rem;
            border-radius: 0.25rem;
            margin-top: 0.5rem;
            width: fit-content;
            align-self: center;
        }
        .credits {
            margin-top: 2rem;
            padding-top: 1rem;
            border-top: 1px solid #e5e7eb;
            color: #6b7280;
            font-size: 0.875rem;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex flex-col items-center p-4">
    <div class="container mx-auto max-w-6xl bg-white shadow-lg rounded-xl p-8 space-y-8">
        <!-- Header -->
        <header class="text-center relative">
            <h1 class="text-5xl font-extrabold text-gray-900 mb-4">Trist's YouTube Client</h1>
            <p class="text-gray-600 text-lg mb-4">Search and browse YouTube content.</p>
            <a href="https://www.youtube.com/@LAtxca" target="_blank" class="inline-flex items-center px-4 py-2 bg-red-600 text-white font-medium rounded-full hover:bg-red-700 transition">
                <svg class="w-5 h-5 mr-2" fill="currentColor" viewBox="0 0 24 24"><path d="M23.498 6.186a3.016 3.016 0 0 0-2.122-2.136C19.505 3.545 12 3.545 12 3.545s-7.505 0-9.377.505A3.017 3.017 0 0[...]
                Visit @LAtxca Channel
            </a>
        </header>

        <!-- Search Bar -->
        <div class="flex flex-col sm:flex-row items-center justify-center gap-4">
            <input
                type="text"
                id="search-input"
                placeholder="Type here to search..."
                class="flex-grow w-full sm:w-auto p-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-lg shadow-sm"
            />
            <button
                id="search-button"
                class="w-full sm:w-auto px-8 py-4 bg-blue-600 text-white font-semibold rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 tr[...]
            >
                <span id="search-button-text">Search</span>
                <div id="loading-spinner" class="spinner ml-2 hidden"></div>
            </button>
        </div>

        <!-- Main Video Player Section -->
        <section id="video-player-section" class="hidden">
            <h2 class="text-3xl font-bold text-gray-800 mb-4 text-center">Now Playing</h2>
            <div class="iframe-container shadow-xl">
                <iframe
                    id="video-player"
                    src=""
                    frameborder="0"
                    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                    allowfullscreen
                ></iframe>
            </div>
            <div class="flex flex-col items-center mt-4">
                <p id="current-video-title" class="text-xl font-semibold text-gray-800 text-center"></p>
                <!-- Fallback link for Error 153 -->
                <a id="external-link" href="#" target="_blank" class="mt-2 text-blue-600 hover:underline text-sm font-medium">
                    Can't see the video? Watch directly on YouTube
                </a>
            </div>
        </section>

        <!-- Video Results Section -->
        <section>
            <h2 class="text-3xl font-bold text-gray-800 mb-6 text-center">Search Results</h2>
            <div id="video-results" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6"></div>
            <p id="no-results-message" class="text-center text-gray-500 text-lg mt-8 hidden">No content found. Try a different search term.</p>
            <p id="error-message" class="text-center text-red-500 text-lg mt-8 hidden"></p>
        </section>

        <!-- Credits Section -->
        <footer class="credits text-center">
            <p class="font-bold text-gray-700">Made by Latxca NOLA</p>
            <p class="text-gray-500">Published by Latxca Games</p>
        </footer>
    </div>

    <script type="module">
        const searchInput = document.getElementById('search-input');
        const searchButton = document.getElementById('search-button');
        const searchButtonText = document.getElementById('search-button-text');
        const loadingSpinner = document.getElementById('loading-spinner');
        const videoResults = document.getElementById('video-results');
        const videoPlayer = document.getElementById('video-player');
        const videoPlayerSection = document.getElementById('video-player-section');
        const currentVideoTitle = document.getElementById('current-video-title');
        const externalLink = document.getElementById('external-link');
        const noResultsMessage = document.getElementById('no-results-message');
        const errorMessage = document.getElementById('error-message');

        // API KEY
        const YOUTUBE_API_KEY = "AIzaSyDhV37HzWpwPK70sU7L8vjI3_ggKWC9jXw"; 

        const mockVideos = [
            { id: 'ynNLs6s_Vsg', type: 'video', title: 'DigBar- Crazy Rap Gay Remix (LATXCA EDIT)', thumbnail: 'https://i.ytimg.com/vi/ynNLs6s_Vsg/mqdefault.jpg', isMusicVideo: true },
            { id: 'C6GsZC0VZe8', type: 'video', title: 'Rich Flex Gay Remix - DigBar (LATXCA SELECTION)', thumbnail: 'https://i.ytimg.com/vi/C6GsZC0VZe8/mqdefault.jpg', isMusicVideo: true },
            { id: 'FhWJHctJOKc', type: 'video', title: 'SHOOTA FLOW 6 GAY REMIX! - LATXCA FAVORITE', thumbnail: 'https://i.ytimg.com/vi/FhWJHctJOKc/mqdefault.jpg', isMusicVideo: true },
            { id: 'HcdKGGs3Vso', type: 'video', title: 'LATXCA PRIDE MEGAMIX - Vol. 6', thumbnail: 'https://i.ytimg.com/vi/HcdKGGs3Vso/mqdefault.jpg', isMusicVideo: true },
            { id: 'E6uWpLz990I', type: 'video', title: 'LATXCA Presents: Best of Digbar Gay Remixes', thumbnail: 'https://i.ytimg.com/vi/E6uWpLz990I/mqdefault.jpg', isMusicVideo: true },
            { id: 'O69V64G5rG0', type: 'video', title: 'Latxca Games - Official Trailer', thumbnail: 'https://i.ytimg.com/vi/O69V64G5rG0/mqdefault.jpg', isMusicVideo: false },
            { id: 'tJjD6_2C9kI', type: 'video', title: 'Travis Scott - SICKO MODE (LATXCA STREAM)', thumbnail: 'https://i.ytimg.com/vi/tJjD6_2C9kI/mqdefault.jpg', isMusicVideo: true },
            { id: 'pR-N793sU9A', type: 'video', title: 'Pop Smoke - Dior (LATXCA NOLA VIBES)', thumbnail: 'https://i.ytimg.com/vi/pR-N793sU9A/mqdefault.jpg', isMusicVideo: true }
        ];

        async function fetchContent(query) {
            errorMessage.classList.add('hidden');
            if (query && YOUTUBE_API_KEY) {
                loadingSpinner.classList.remove('hidden');
                searchButtonText.textContent = 'Searching...';
                try {
                    const url = `https://www.googleapis.com/youtube/v3/search?part=snippet&type=video&q=${encodeURIComponent(query)}&maxResults=12&key=${YOUTUBE_API_KEY}`;
                    const res = await fetch(url);
                    const data = await res.json();
                    if (data.error) throw new Error(data.error.message);
                    if (data.items && data.items.length > 0) {
                        return data.items.map(item => ({
                            type: 'video',
                            title: item.snippet.title,
                            videoId: item.id.videoId,
                            thumbnail: item.snippet.thumbnails.medium.url,
                            description: item.snippet.description
                        }));
                    }
                } catch (e) {
                    console.error("Search error:", e);
                    errorMessage.textContent = "Search failed. Showing local results.";
                    errorMessage.classList.remove('hidden');
                } finally {
                    loadingSpinner.classList.add('hidden');
                    searchButtonText.textContent = 'Search';
                }
            }
            const q = query.toLowerCase();
            return q ? mockVideos.filter(v => v.title.toLowerCase().includes(q)) : mockVideos;
        }

        function displayContent(contents) {
            videoResults.innerHTML = '';
            if (!contents || contents.length === 0) {
                noResultsMessage.classList.remove('hidden');
                return;
            }
            noResultsMessage.classList.add('hidden');

            contents.forEach(video => {
                const card = document.createElement('div');
                card.className = 'video-thumbnail bg-white rounded-lg shadow-md p-4 flex flex-col items-center text-center hover:bg-gray-50';
                card.innerHTML = `
                    <img src="${video.thumbnail}" class="w-full h-auto rounded-md mb-3 aspect-video object-cover">
                    <h3 class="text-sm font-semibold text-gray-800 line-clamp-2">${video.title}</h3>
                    ${video.isMusicVideo ? '<p class="music-warning">🎶 Music Video</p>' : ''}
                `;

                card.onclick = () => {
                    const id = video.videoId || video.id;
                    // FIX: Using the minimal embed URL to avoid "Error 153" config issues
                    // origin parameter helps with domain validation
                    videoPlayer.src = `https://www.youtube.com/embed/${id}?autoplay=1&rel=0&modestbranding=1`;
                    currentVideoTitle.textContent = video.title;
                    externalLink.href = `https://www.youtube.com/watch?v=${id}`;
                    videoPlayerSection.classList.remove('hidden');
                    videoPlayerSection.scrollIntoView({ behavior: 'smooth' });
                };
                videoResults.appendChild(card);
            });
        }

        async function handleSearch() {
            const val = searchInput.value.trim();
            const results = await fetchContent(val);
            displayContent(results);
        }

        searchButton.addEventListener('click', handleSearch);
        searchInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') handleSearch();
        });

        window.addEventListener('DOMContentLoaded', () => {
            fetchContent("").then(displayContent);
        });
    </script>
</body>
</html>
