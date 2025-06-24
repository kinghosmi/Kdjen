<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GeoLink Tracker - Location Tracking Links</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/leaflet.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/leaflet.css" />
    <style>
        .gradient-bg {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        .map-container {
            height: 400px;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }
        .link-card {
            transition: all 0.3s ease;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }
        .link-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
        }
        .glow {
            animation: glow 2s infinite alternate;
        }
        @keyframes glow {
            from {
                box-shadow: 0 0 5px rgba(99, 102, 241, 0.5);
            }
            to {
                box-shadow: 0 0 20px rgba(99, 102, 241, 0.8);
            }
        }
    </style>
</head>
<body class="min-h-screen gradient-bg">
    <div class="container mx-auto px-4 py-12">
        <!-- Header -->
        <header class="text-center mb-12">
            <h1 class="text-4xl md:text-5xl font-bold text-white mb-4">GeoLink Tracker</h1>
            <p class="text-white text-xl opacity-90">Create links that track visitor locations in real-time</p>
        </header>

        <!-- Main Content -->
        <div class="max-w-6xl mx-auto">
            <!-- Create Link Section -->
            <div class="bg-white rounded-xl p-6 link-card mb-8 glow">
                <h2 class="text-2xl font-bold text-gray-800 mb-4">Create New Tracking Link</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                    <div>
                        <label for="destination" class="block text-gray-700 mb-2">Destination URL*</label>
                        <input type="url" id="destination" placeholder="https://example.com" 
                               class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent" required>
                    </div>
                    <div>
                        <label for="linkName" class="block text-gray-700 mb-2">Campaign Name</label>
                        <input type="text" id="linkName" placeholder="Summer Sale 2023" 
                               class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent">
                    </div>
                </div>
                <div class="mb-4">
                    <label class="block text-gray-700 mb-2">Tracking Options</label>
                    <div class="flex flex-wrap gap-4">
                        <label class="inline-flex items-center">
                            <input type="checkbox" id="trackLocation" class="rounded text-indigo-600" checked>
                            <span class="ml-2">Geolocation</span>
                        </label>
                        <label class="inline-flex items-center">
                            <input type="checkbox" id="trackDevice" class="rounded text-indigo-600" checked>
                            <span class="ml-2">Device Info</span>
                        </label>
                        <label class="inline-flex items-center">
                            <input type="checkbox" id="trackReferrer" class="rounded text-indigo-600">
                            <span class="ml-2">Referrer</span>
                        </label>
                    </div>
                </div>
                <button id="generateBtn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-4 rounded-lg transition duration-200">
                    Generate Tracking Link
                </button>
            </div>

            <!-- Generated Link Section -->
            <div id="generatedLinkSection" class="bg-white rounded-xl p-6 link-card mb-8 hidden">
                <h2 class="text-2xl font-bold text-gray-800 mb-4">Your Tracking Link</h2>
                <div class="mb-4">
                    <label class="block text-gray-700 mb-2">Share this link:</label>
                    <div class="flex">
                        <input id="trackingLink" type="text" readonly 
                               class="flex-grow px-4 py-2 border border-gray-300 rounded-l-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent">
                        <button id="copyBtn" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-r-lg transition duration-200">
                            Copy
                        </button>
                    </div>
                </div>
                <div class="mb-4">
                    <label class="block text-gray-700 mb-2">Tracking Dashboard:</label>
                    <input id="dashboardLink" type="text" readonly 
                           class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent">
                </div>
                <div class="text-sm text-gray-500">
                    <p>When someone clicks your link, their location and device info will be recorded.</p>
                    <p>You can view the tracking data using the dashboard link above.</p>
                </div>
            </div>

            <!-- Tracking Dashboard -->
            <div id="dashboardSection" class="bg-white rounded-xl p-6 link-card hidden">
                <h2 class="text-2xl font-bold text-gray-800 mb-4">Tracking Dashboard</h2>
                <div class="mb-4">
                    <div class="flex justify-between items-center mb-2">
                        <h3 class="text-lg font-semibold text-gray-700">Campaign: <span id="campaignName"></span></h3>
                        <div class="text-indigo-600 font-medium">Total Clicks: <span id="totalClicks">0</span></div>
                    </div>
                    <div class="map-container mb-4" id="map"></div>
                </div>
                <div class="overflow-x-auto">
                    <table class="min-w-full bg-white rounded-lg overflow-hidden">
                        <thead class="bg-gray-100">
                            <tr>
                                <th class="py-3 px-4 text-left text-gray-700">Time</th>
                                <th class="py-3 px-4 text-left text-gray-700">Location</th>
                                <th class="py-3 px-4 text-left text-gray-700">Device</th>
                                <th class="py-3 px-4 text-left text-gray-700">IP</th>
                            </tr>
                        </thead>
                        <tbody id="trackingData" class="divide-y divide-gray-200">
                            <!-- Tracking data will be inserted here -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Initialize variables
        let map;
        let trackingLinks = JSON.parse(localStorage.getItem('trackingLinks')) || [];
        let currentLinkId = null;

        // DOM Elements
        const generateBtn = document.getElementById('generateBtn');
        const generatedLinkSection = document.getElementById('generatedLinkSection');
        const dashboardSection = document.getElementById('dashboardSection');
        const trackingLinkInput = document.getElementById('trackingLink');
        const dashboardLinkInput = document.getElementById('dashboardLink');
        const copyBtn = document.getElementById('copyBtn');
        const campaignNameSpan = document.getElementById('campaignName');
        const totalClicksSpan = document.getElementById('totalClicks');
        const trackingDataTbody = document.getElementById('trackingData');

        // Generate tracking link
        generateBtn.addEventListener('click', () => {
            const destination = document.getElementById('destination').value;
            const linkName = document.getElementById('linkName').value || 'Untitled Campaign';
            const trackLocation = document.getElementById('trackLocation').checked;
            const trackDevice = document.getElementById('trackDevice').checked;
            const trackReferrer = document.getElementById('trackReferrer').checked;

            if (!destination) {
                Swal.fire('Error', 'Please enter a destination URL', 'error');
                return;
            }

            // Generate unique ID for this tracking link
            const linkId = 'gl-' + Math.random().toString(36).substr(2, 9);
            
            // Create tracking link
            const trackingLink = `${window.location.origin}${window.location.pathname}?id=${linkId}&redirect=${encodeURIComponent(destination)}`;
            const dashboardLink = `${window.location.origin}${window.location.pathname}?dashboard=${linkId}`;

            // Save to local storage
            const newLink = {
                id: linkId,
                name: linkName,
                destination: destination,
                createdAt: new Date().toISOString(),
                trackLocation,
                trackDevice,
                trackReferrer,
                clicks: []
            };

            trackingLinks.push(newLink);
            localStorage.setItem('trackingLinks', JSON.stringify(trackingLinks));

            // Show generated links
            trackingLinkInput.value = trackingLink;
            dashboardLinkInput.value = dashboardLink;
            generatedLinkSection.classList.remove('hidden');

            // Show dashboard if this is the current link
            if (window.location.search.includes(`dashboard=${linkId}`)) {
                showDashboard(linkId);
            }

            // Scroll to generated link section
            generatedLinkSection.scrollIntoView({ behavior: 'smooth' });
        });

        // Copy tracking link
        copyBtn.addEventListener('click', () => {
            trackingLinkInput.select();
            document.execCommand('copy');
            Swal.fire('Copied!', 'Tracking link copied to clipboard', 'success');
        });

        // Check URL parameters on page load
        document.addEventListener('DOMContentLoaded', () => {
            const urlParams = new URLSearchParams(window.location.search);
            const linkId = urlParams.get('id');
            const redirectUrl = urlParams.get('redirect');
            const dashboardId = urlParams.get('dashboard');

            if (linkId && redirectUrl) {
                // This is a tracking link click - capture data and redirect
                handleTrackingClick(linkId, redirectUrl);
            } else if (dashboardId) {
                // Show dashboard for this tracking link
                showDashboard(dashboardId);
            }
        });

        // Handle tracking link click
        function handleTrackingClick(linkId, redirectUrl) {
            const linkData = trackingLinks.find(link => link.id === linkId);
            if (!linkData) {
                window.location.href = redirectUrl;
                return;
            }

            const clickData = {
                timestamp: new Date().toISOString(),
                ip: 'Unknown', // Would be captured server-side in a real app
                userAgent: navigator.userAgent,
                referrer: document.referrer || 'Direct'
            };

            // Get geolocation if enabled and available
            if (linkData.trackLocation && navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        clickData.location = {
                            lat: position.coords.latitude,
                            lng: position.coords.longitude,
                            accuracy: position.coords.accuracy
                        };
                        saveClickAndRedirect(linkId, clickData, redirectUrl);
                    },
                    (error) => {
                        console.error('Geolocation error:', error);
                        saveClickAndRedirect(linkId, clickData, redirectUrl);
                    }
                );
            } else {
                saveClickAndRedirect(linkId, clickData, redirectUrl);
            }
        }

        function saveClickAndRedirect(linkId, clickData, redirectUrl) {
            const linkIndex = trackingLinks.findIndex(link => link.id === linkId);
            if (linkIndex !== -1) {
                trackingLinks[linkIndex].clicks.push(clickData);
                localStorage.setItem('trackingLinks', JSON.stringify(trackingLinks));
            }
            window.location.href = redirectUrl;
        }

        // Show dashboard for a tracking link
        function showDashboard(linkId) {
            const linkData = trackingLinks.find(link => link.id === linkId);
            if (!linkData) {
                Swal.fire('Error', 'Tracking link not found', 'error');
                return;
            }

            currentLinkId = linkId;
            campaignNameSpan.textContent = linkData.name;
            totalClicksSpan.textContent = linkData.clicks.length;

            // Initialize map
            if (!map) {
                map = L.map('map').setView([20, 0], 2);
                L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                    attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
                }).addTo(map);
            }

            // Clear existing markers and data
            map.eachLayer(layer => {
                if (layer instanceof L.Marker) {
                    map.removeLayer(layer);
                }
            });
            trackingDataTbody.innerHTML = '';

            // Add markers for each click with location
            linkData.clicks.forEach((click, index) => {
                if (click.location) {
                    const marker = L.marker([click.location.lat, click.location.lng]).addTo(map);
                    marker.bindPopup(`<b>Click #${index + 1}</b><br>${new Date(click.timestamp).toLocaleString()}`);
                    
                    // Adjust map view to show all markers
                    if (index === 0) {
                        map.setView([click.location.lat, click.location.lng], 5);
                    } else {
                        map.fitBounds([
                            ...map.getBounds().getSouthWest().toArray(),
                            [click.location.lat, click.location.lng]
                        ]);
                    }
                }

                // Add to tracking table
                const row = document.createElement('tr');
                row.className = index % 2 === 0 ? 'bg-gray-50' : 'bg-white';
                row.innerHTML = `
                    <td class="py-3 px-4">${new Date(click.timestamp).toLocaleString()}</td>
                    <td class="py-3 px-4">${click.location ? `${click.location.lat.toFixed(4)}, ${click.location.lng.toFixed(4)}` : 'Unknown'}</td>
                    <td class="py-3 px-4">${getDeviceInfo(click.userAgent)}</td>
                    <td class="py-3 px-4">${click.ip}</td>
                `;
                trackingDataTbody.appendChild(row);
            });

            // Show dashboard sections
            generatedLinkSection.classList.add('hidden');
            dashboardSection.classList.remove('hidden');
            dashboardSection.scrollIntoView({ behavior: 'smooth' });
        }

        // Helper function to parse device info from user agent
        function getDeviceInfo(userAgent) {
            const isMobile = /Mobile|Android|iPhone|iPad|iPod/i.test(userAgent);
            const isWindows = /Windows/i.test(userAgent);
            const isMac = /Macintosh|Mac OS X/i.test(userAgent);
            const isLinux = /Linux/i.test(userAgent);
            
            let os = 'Unknown OS';
            if (isWindows) os = 'Windows';
            else if (isMac) os = 'Mac';
            else if (isLinux) os = 'Linux';
            
            let deviceType = isMobile ? 'Mobile' : 'Desktop';
            
            return `${deviceType} (${os})`;
        }
    </script>
</body>
</html>
