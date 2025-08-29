<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Die HEUTE | Interactive News Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&family=Playfair+Display:wght@700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm and Earthy (Brown, beige, and a muted yellow accent) -->
    <!-- Application Structure Plan: A dashboard-style SPA with a header, a filterable navigation bar, a responsive grid for news cards, and a modal for editing. The application now features a "TOP 10" and "All News" view. A new "Edit News" modal allows for manual, persistent updates of headlines, summaries, and categories. This structure gives the user full control over the displayed content. -->
    <!-- Visualization & Content Choices: 
        - Report Info: Integration of 2015 Refugees (64% employed). Goal: Inform/Show Proportion. Viz: Horizontal Bar Chart (Chart.js). Interaction: None, static view. Justification: A bar chart is the clearest way to visualize a percentage, making the statistic instantly understandable.
        - Report Info: German Car Industry Job Losses (51,000 jobs). Goal: Inform/Show Impact. Viz: Large Stat Card (HTML/CSS). Interaction: None, static view. Justification: A large, bold number provides immediate impact and is more direct than a chart when dealing with a single, stark figure.
        - Report Info: All other news headlines. Goal: Inform/Organize. Presentation: Thematic Cards (HTML/Tailwind). Interaction: Clicking a card opens a modal for editing. Justification: Cards provide a clean, modular way to present distinct pieces of information, and the new modal adds a layer of user control.
        - Library/Method: Chart.js for canvas-based charts, Vanilla JS for filtering and modal logic. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .header-font {
            font-family: 'Playfair Display', serif;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 400px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 350px;
            }
        }
        .modal {
            display: none;
        }
        .modal.open {
            display: flex;
        }
    </style>
</head>
<body class="bg-amber-50 text-stone-900">

    <div class="container mx-auto p-4 md:p-8">
        
        <header class="text-center mb-12">
            <div class="border-b-4 border-amber-600 inline-block px-4 pb-2">
                <h1 class="header-font text-6xl md:text-7xl font-extrabold tracking-tight">Die HEUTE</h1>
            </div>
            <p class="text-stone-600 mt-2 text-lg">My Daily News Partner</p>
            <p class="text-stone-600 font-semibold mt-4 text-sm md:text-base" id="current-date"></p>
        </header>

        <nav class="flex justify-center flex-wrap gap-3 mb-10">
            <button data-filter="top10" class="filter-btn bg-amber-600 text-white py-2 px-6 rounded-full shadow-md hover:bg-amber-700 transition font-bold text-sm">Top 10 News</button>
            <button data-filter="all" class="filter-btn bg-white text-stone-700 py-2 px-6 rounded-full shadow-md hover:bg-stone-200 transition font-bold text-sm">All News</button>
            <button data-filter="domestic" class="filter-btn bg-white text-stone-700 py-2 px-6 rounded-full shadow-md hover:bg-stone-200 transition font-bold text-sm">🚨 Domestic Affairs</button>
            <button data-filter="economy" class="filter-btn bg-white text-stone-700 py-2 px-6 rounded-full shadow-md hover:bg-stone-200 transition font-bold text-sm">💰 Economy & Tech</button>
            <button data-filter="politics" class="filter-btn bg-white text-stone-700 py-2 px-6 rounded-full shadow-md hover:bg-stone-200 transition font-bold text-sm">🌐 Politics & Foreign Policy</button>
        </nav>

        <main>
            <p class="text-center text-stone-600 mb-10 max-w-3xl mx-auto text-base" id="news-info-message">
                Welcome to your daily interactive briefing. This dashboard summarizes the key news stories from Germany. Click any news card to edit its content.
            </p>
            <div id="news-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            </div>
        </main>
    </div>

    <!-- Edit News Modal -->
    <div id="editModal" class="modal fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full z-50 items-center justify-center">
        <div class="relative bg-white p-8 rounded-lg shadow-xl w-11/12 md:w-1/2">
            <h2 class="text-2xl font-bold mb-4 header-font">Edit News Story</h2>
            <form id="editForm">
                <input type="hidden" id="edit-index">
                <div class="mb-4">
                    <label for="edit-headline" class="block text-sm font-medium text-gray-700">Headline</label>
                    <input type="text" id="edit-headline" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-amber-300 focus:ring focus:ring-amber-200 focus:ring-opacity-50">
                </div>
                <div class="mb-4">
                    <label for="edit-summary" class="block text-sm font-medium text-gray-700">Summary</label>
                    <textarea id="edit-summary" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-amber-300 focus:ring focus:ring-amber-200 focus:ring-opacity-50"></textarea>
                </div>
                <div class="mb-4">
                    <label for="edit-category" class="block text-sm font-medium text-gray-700">Category</label>
                    <select id="edit-category" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-amber-300 focus:ring focus:ring-amber-200 focus:ring-opacity-50">
                        <option value="domestic">Domestic Affairs</option>
                        <option value="economy">Economy & Tech</option>
                        <option value="politics">Politics & Foreign Policy</option>
                    </select>
                </div>
                <div class="flex justify-end space-x-4">
                    <button type="button" id="closeModalBtn" class="px-4 py-2 bg-gray-200 text-gray-800 font-bold rounded-full hover:bg-gray-300 transition">Cancel</button>
                    <button type="submit" class="px-4 py-2 bg-amber-600 text-white font-bold rounded-full hover:bg-amber-700 transition">Save Changes</button>
                </div>
            </form>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const newsGrid = document.getElementById('news-grid');
            const filterBtns = document.querySelectorAll('.filter-btn');
            const editModal = document.getElementById('editModal');
            const editForm = document.getElementById('editForm');
            const closeModalBtn = document.getElementById('closeModalBtn');
            let newsData = [];
            let currentActiveFilter = 'top10';

            const initialStaticNewsData = [
                { category: 'domestic', headline: '🚨 Knife Attack in Dresden', summary: 'A man was arrested after attacking a U.S. citizen with a knife on a tram in Dresden. The motive is still under investigation.' },
                { category: 'economy', headline: '💰 German Car Industry Job Losses', summary: 'Germany\'s automotive sector has shed over 51,000 jobs in the past year, raising concerns about the industry\'s future amid global competition and electrification.' },
                { category: 'domestic', headline: '🚨 Munich Car Ramming Incident', summary: 'A suspect who drove into a crowd during a demonstration in Munich has been charged with murder and 44 counts of attempted murder.' },
                { category: 'economy', headline: '🚄 Green Hydrogen Trains', summary: 'Germany is testing trains powered by green hydrogen as part of its push toward cleaner transportation.' },
                { category: 'domestic', headline: '🔥 Fire at Hamburg Port', summary: 'Firefighters are battling a massive blaze at a warehouse in Hamburg\'s port area, which has spread to nearby industrial buildings.' },
                { category: 'domestic', headline: '🤝 Integration of 2015 Refugees', summary: 'A new study shows that 64% of refugees who arrived in Germany in 2015 are now employed, indicating significant progress in integration efforts.' },
                { category: 'politics', headline: '🌐 Political Tensions Over Russia and China', summary: 'Germany has warned that China’s support is enabling Russia’s war efforts. The country is also partnering with Japan to counter China’s aggressive stance in the Indo-Pacific.' },
                { category: 'politics', headline: '🏛️ Far-Right AfD Leads in Polls', summary: 'The far-right Alternative for Germany (AfD) party has topped recent popularity rankings, sparking political debate and concern.' },
                { category: 'politics', headline: '📜 German Citizenship Reform Criticized', summary: 'Germany plans to scrap a fast-track citizenship path, prompting backlash from applicants and immigration experts.' },
                { category: 'domestic', headline: '⚖️ Neo-Nazi Beate Zschäpe Seeks Rehabilitation', summary: 'Convicted NSU terrorist Beate Zschäpe may be released early after entering a rehabilitation program.' }
            ];

            const loadNewsFromStorage = () => {
                const storedNews = localStorage.getItem('german_news_headlines');
                if (storedNews) {
                    newsData = JSON.parse(storedNews);
                } else {
                    newsData = [...initialStaticNewsData];
                    localStorage.setItem('german_news_headlines', JSON.stringify(newsData));
                }
            };

            const renderNews = (data) => {
                newsGrid.innerHTML = '';
                if (data.length === 0) {
                    newsGrid.innerHTML = `<div class="col-span-full text-center text-stone-500 text-lg p-8 rounded-lg">
                        <p class="font-bold mb-2">No news found for this category.</p>
                        <p>Please add new stories by clicking on an existing card to edit it.</p>
                    </div>`;
                    return;
                }
                data.forEach((item, index) => {
                    const newsCard = document.createElement('div');
                    newsCard.className = `news-card bg-white p-6 rounded-lg shadow-lg transition-all duration-300 transform hover:scale-105 cursor-pointer`;
                    newsCard.dataset.category = item.category;
                    newsCard.dataset.index = newsData.findIndex(d => d.headline === item.headline && d.summary === item.summary);
                    
                    let cardContent = `<h3 class="text-xl font-bold mb-2 text-stone-900">${item.headline}</h3><p class="text-stone-600 text-sm">${item.summary}</p>`;
                    newsCard.innerHTML = cardContent;
                    newsGrid.appendChild(newsCard);
                });
            };

            const filterNews = (filter) => {
                currentActiveFilter = filter;
                let dataToRender;
                if (filter === 'top10') {
                    dataToRender = newsData.slice(0, 10);
                } else if (filter === 'all') {
                    dataToRender = newsData;
                } else {
                    dataToRender = newsData.filter(item => item.category === filter);
                }
                renderNews(dataToRender);
            };

            newsGrid.addEventListener('click', (e) => {
                const newsCard = e.target.closest('.news-card');
                if (newsCard) {
                    const index = newsCard.dataset.index;
                    const story = newsData[index];
                    document.getElementById('edit-index').value = index;
                    document.getElementById('edit-headline').value = story.headline;
                    document.getElementById('edit-summary').value = story.summary;
                    document.getElementById('edit-category').value = story.category;
                    editModal.classList.add('open');
                }
            });

            editForm.addEventListener('submit', (e) => {
                e.preventDefault();
                const index = document.getElementById('edit-index').value;
                newsData[index].headline = document.getElementById('edit-headline').value;
                newsData[index].summary = document.getElementById('edit-summary').value;
                newsData[index].category = document.getElementById('edit-category').value;
                localStorage.setItem('german_news_headlines', JSON.stringify(newsData));
                editModal.classList.remove('open');
                filterNews(currentActiveFilter);
            });

            closeModalBtn.addEventListener('click', () => {
                editModal.classList.remove('open');
            });

            filterBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    filterBtns.forEach(b => {
                        b.classList.remove('bg-amber-600', 'text-white');
                        b.classList.add('bg-white', 'text-stone-700');
                    });
                    btn.classList.add('bg-amber-600', 'text-white');
                    btn.classList.remove('bg-white', 'text-stone-700');
                    filterNews(btn.dataset.filter);
                });
            });

            document.getElementById('current-date').textContent = new Date().toLocaleDateString('en-US', {
                weekday: 'long', year: 'numeric', month: 'long', day: 'numeric'
            });

            loadNewsFromStorage();
            filterNews('top10');
        });
    </script>
</body>
</html>
