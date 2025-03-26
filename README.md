# webproject
webproject description
Task 1 :
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Movie & TV Show Search</title>
    <style>
        body {
            font-family: sans-serif;
            margin: 20px;
        }
        #search-container {
            margin-bottom: 20px;
        }
        #results-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
        }
        .result-item {
            border: 1px solid #ddd;
            padding: 10px;
            text-align: center;
        }
        .result-item img {
            max-width: 100%;
            height: auto;
        }
        .pagination {
            display: flex;
            justify-content: center;
            margin-top: 20px;
        }
        .pagination button {
            margin: 0 5px;
            padding: 8px 15px;
            cursor: pointer;
        }
        .skeleton-loader {
            background: linear-gradient(90deg, #eee 25%, #f5f5f5 50%, #eee 75%);
            animation: skeleton-loading 1s linear infinite;
            height: 200px;
            width: 100%;
            margin-bottom: 10px;
        }

        @keyframes skeleton-loading {
            0% { background-position: 200% 0; }
            100% { background-position: -200% 0; }
        }

        .error-message {
          color: red;
          text-align: center;
        }

    </style>
</head>
<body>

    <div id="search-container">
        <input type="text" id="search-input" placeholder="Search movies, shows, actors...">
        <button id="search-button">Search</button>
    </div>

    <div id="results-container">
        </div>

    <div id="pagination-container" class="pagination">
        </div>

    <script>
        const apiKey = 'YOUR_TMDB_API_KEY'; // Replace with your TMDB API key
        const searchInput = document.getElementById('search-input');
        const searchButton = document.getElementById('search-button');
        const resultsContainer = document.getElementById('results-container');
        const paginationContainer = document.getElementById('pagination-container');

        let currentPage = 1;
        let totalPages = 1;
        let currentQuery = '';
        let cache = {};

        async function fetchWithRetry(url, options = {}, retries = 3, backoff = 300) {
            try {
                const response = await fetch(url, options);
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                return response.json();
            } catch (error) {
                if (retries > 0) {
                    await new Promise(resolve => setTimeout(resolve, backoff));
                    return fetchWithRetry(url, options, retries - 1, backoff * 2);
                }
                throw error;
            }
        }

        async function search(query, page = 1) {
            currentQuery = query;
            currentPage = page;

            if (cache[query] && cache[query][page]) {
                displayResults(cache[query][page].results);
                updatePagination();
                return;
            }

            resultsContainer.innerHTML = '<div class="skeleton-loader"></div><div class="skeleton-loader"></div><div class="skeleton-loader"></div><div class="skeleton-loader"></div>';

            try {
                const url = `https://api.themoviedb.org/3/search/multi?api_key=${apiKey}&query=${encodeURIComponent(query)}&page=${page}`;
                const data = await fetchWithRetry(url);

                if (!cache[query]) {
                    cache[query] = {};
                }
                cache[query][page] = data;

                totalPages = data.total_pages;
                displayResults(data.results);
                updatePagination();

            } catch (error) {
                console.error('Search error:', error);
                resultsContainer.innerHTML = '<div class="error-message">Search failed. Please try again later.</div>';
                paginationContainer.innerHTML = '';
            }
        }

        function displayResults(results) {
            resultsContainer.innerHTML = '';
            results.forEach(result => {
                const item = document.createElement('div');
                item.classList.add('result-item');

                let imageUrl = 'placeholder.png';
                if (result.poster_path) {
                    imageUrl = `https://image.tmdb.org/t/p/w200${result.poster_path}`;
                } else if (result.profile_path) {
                    imageUrl = `https://image.tmdb.org/t/p/w200${result.profile_path}`;
                }

                item.innerHTML = `
                    <img src="${imageUrl}" alt="${result.title || result.name}">
                    <h3>${result.title || result.name}</h3>
                    <p>${result.media_type}</p>
                `;
                resultsContainer.appendChild(item);
            });
        }

        function updatePagination() {
            paginationContainer.innerHTML = '';
            if(totalPages > 1){
              if (currentPage > 1) {
                  const prevButton = document.createElement('button');
                  prevButton.textContent = 'Previous';
                  prevButton.addEventListener('click', () => search(currentQuery, currentPage - 1));
                  paginationContainer.appendChild(prevButton);
              }

              if (currentPage < totalPages) {
                  const nextButton = document.createElement('button');
                  nextButton.textContent = 'Next';
                  nextButton.addEventListener('click', () => search(currentQuery, currentPage + 1));
                  paginationContainer.appendChild(nextButton);
              }
            }

        }

        searchButton.addEventListener('click', () => {
            const query = searchInput.value.trim();
            if (query) {
                search(query);
            }
        });

    </script>
</body>
</html>

Task 2 :

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Actor Profile</title>
    <style>
        body {
            font-family: sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }

        .actor-profile {
            max-width: 800px;
            margin: 20px auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }

        .actor-profile img {
            max-width: 200px;
            border-radius: 50%;
            margin-bottom: 20px;
        }

        .actor-profile h1 {
            margin-bottom: 10px;
        }

        .movie-list {
            margin-top: 20px;
        }

        .movie-list ul {
            list-style: none;
            padding: 0;
        }

        .movie-list li {
            margin-bottom: 10px;
            padding: 10px;
            background-color: #f9f9f9;
            border-radius: 4px;
        }
    </style>
</head>
<body>

    <div class="actor-profile">
        <img id="actor-image" src="" alt="Actor Profile Image">
        <h1 id="actor-name"></h1>
        <p id="actor-bio"></p>

        <div class="movie-list">
            <h2>Movies</h2>
            <ul id="actor-movies">
                </ul>
        </div>
    </div>

    <script>
        // Mock API data (replace with your actual API or Firestore fetch)
        const mockActorData = {
            imageUrl: "https://via.placeholder.com/200", // Replace with real image URL
            name: "Example Actor",
            bio: "This is a brief biography of the actor. Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
            movies: [
                { title: "Movie 1", year: 2020 },
                { title: "Movie 2", year: 2021 },
                { title: "Movie 3", year: 2022 }
            ]
        };

        function populateActorProfile(actorData) {
            document.getElementById("actor-image").src = actorData.imageUrl;
            document.getElementById("actor-name").textContent = actorData.name;
            document.getElementById("actor-bio").textContent = actorData.bio;

            const moviesList = document.getElementById("actor-movies");
            actorData.movies.forEach(movie => {
                const listItem = document.createElement("li");
                listItem.textContent = `${movie.title} (${movie.year})`;
                moviesList.appendChild(listItem);
            });
        }

        // Simulate API fetch (replace with actual fetch)
        function fetchActorData() {
            return new Promise((resolve) => {
                setTimeout(() => {
                    resolve(mockActorData);
                }, 500); // Simulate network delay
            });
        }

        // Main function to load data and populate the profile
        async function loadActorProfile() {
            const actorData = await fetchActorData();
            populateActorProfile(actorData);
        }

        // Load the profile when the page loads
        loadActorProfile();
    </script>

</body>
</html>

Task 3 :

<!DOCTYPE html>
<html>
<head>
    <title>Movie Collection</title>
    <style>
        body { font-family: sans-serif; }
        #view-toggle { margin-bottom: 20px; }
        #view-toggle button { padding: 8px 15px; }
        #movie-container { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; }
        #movie-container.list-view { display: block; }
        .movie-item { border: 1px solid #ddd; padding: 10px; transition: all 0.3s ease; }
        .movie-item.list-view { display: flex; align-items: flex-start; }
        .movie-item img { max-width: 100%; height: auto; }
        .movie-item.list-view img { max-width: 150px; margin-right: 20px; }
        .movie-details { flex-grow: 1; }
        .movie-details h3 { margin-top: 0; }
        .movie-genres { margin-bottom: 10px; }
        .movie-genres span { background-color: #f0f0f0; padding: 3px 8px; margin-right: 5px; border-radius: 5px; }
        .movie-synopsis { border-top: 1px solid #eee; margin-top: 10px; padding-top: 10px; max-height: 0; overflow: hidden; transition: max-height 0.3s ease; }
        .movie-synopsis.expanded { max-height: 500px; }
        .masonry {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            grid-auto-rows: 10px;
            gap: 20px;
        }

        .masonry .movie-item{
            grid-row: span 1;
        }

        @media (max-width: 600px) {
            #movie-container.list-view .movie-item{
                flex-direction: column;
            }
            #movie-container.list-view .movie-item img{
                margin-right: 0;
                margin-bottom:10px;
                max-width: 100%;
            }
        }

    </style>
</head>
<body>
    <div id="view-toggle">
        <button id="grid-view-button">Grid View</button>
        <button id="list-view-button">List View</button>
    </div>
    <div id="movie-container" class="masonry">
        </div>

    <script>
        const movieData = [
            {
                title: 'Movie 1',
                rating: 8.5,
                description: 'A thrilling adventure...',
                genres: ['Action', 'Adventure'],
                poster: 'https://via.placeholder.com/200x300',
                synopsis: 'Long synopsis of movie 1...'
            },
            {
                title: 'Movie 2',
                rating: 7.8,
                description: 'A romantic comedy...',
                genres: ['Comedy', 'Romance'],
                poster: 'https://via.placeholder.com/200x320',
                synopsis: 'Long synopsis of movie 2...'
            },
            {
                title: 'Movie 3',
                rating: 9.2,
                description: 'A gripping drama...',
                genres: ['Drama'],
                poster: 'https://via.placeholder.com/200x280',
                synopsis: 'Long synopsis of movie 3...'
            },
            {
                title: 'Movie 4',
                rating: 6.5,
                description: 'A sci-fi thriller...',
                genres: ['Sci-Fi', 'Thriller'],
                poster: 'https://via.placeholder.com/200x350',
                synopsis: 'Long synopsis of movie 4...'
            },
            {
                title: 'Movie 5',
                rating: 8.0,
                description: 'An animated family film...',
                genres: ['Animation', 'Family'],
                poster: 'https://via.placeholder.com/200x310',
                synopsis: 'Long synopsis of movie 5...'
            },
            // Add more movie data as needed
        ];

        const movieContainer = document.getElementById('movie-container');
        const gridViewButton = document.getElementById('grid-view-button');
        const listViewButton = document.getElementById('list-view-button');

        function displayMovies(viewType) {
            movieContainer.innerHTML = '';
            movieContainer.classList.remove('list-view', 'masonry');
            if (viewType === 'list') {
                movieContainer.classList.add('list-view');
            } else {
                movieContainer.classList.add('masonry');
            }

            movieData.forEach(movie => {
                const movieItem = document.createElement('div');
                movieItem.classList.add('movie-item');
                if (viewType === 'list') {
                    movieItem.classList.add('list-view');
                    movieItem.innerHTML = `
                        <img src="${movie.poster}" alt="${movie.title}">
                        <div class="movie-details">
                            <h3>${movie.title}</h3>
                            <p>Rating: ${movie.rating}</p>
                            <p>${movie.description}</p>
                            <div class="movie-genres">${movie.genres.map(genre => `<span>${genre}</span>`).join('')}</div>
                            <div class="movie-synopsis">${movie.synopsis}</div>
                        </div>
                    `;
                    movieItem.querySelector('.movie-synopsis').addEventListener('click', function() {
                        this.classList.toggle('expanded');
                    });

                } else {
                    movieItem.innerHTML = `<img src="${movie.poster}" alt="${movie.title}">`;
                    movieItem.style.gridRow = `span ${Math.ceil(parseFloat(movie.poster.split('x')[1]) / 200)}`;
                }
                movieContainer.appendChild(movieItem);
            });
        }

        gridViewButton.addEventListener('click', () => displayMovies('grid'));
        listViewButton.addEventListener('click', () => displayMovies('list'));

        displayMovies('grid'); // Initial display in grid view
    </script>
</body>
</html>
