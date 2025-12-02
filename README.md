# Music Week Archive

A comprehensive web-based music analytics and exploration tool designed for browsing, searching, and visualizing large music datasets. Built with vanilla JavaScript and designed for performance at scale.

## Overview

The Music Week Archive is a single-page application that transforms CSV music data into an interactive exploration experience. Users can browse music organized by multiple dimensions—tracks, artists, albums, genres, contributors, partners, and playlists—with advanced filtering, search, and navigation capabilities.

## Core Features

### Data Views

**Overview Tab** - Dashboard displaying key statistics including total tracks, unique artists, album count, genre diversity, total duration, and contributor count. Includes top 10 charts for artists and genres.

**Tracks Tab** - Full track catalog with year-based filtering. Each track displays album artwork (with lazy loading), artist information, genre tags, contributor details, and direct links to Spotify. Supports search filtering and pagination for large datasets.

**Albums Tab** - Albums organized with associated artwork, track counts, contributor information, and genre classifications. Year filters available for temporal browsing. Click-through capability to view all tracks within an album.

**Artists Tab** - Artist rankings by appearance frequency. Displays top albums per artist, contributor network, and years active. Search functionality and year filtering included.

**Genres Tab** - Genre hierarchy with track counts and popularity percentages. Shows top contributing artists and contributors for each genre. Supports drilling down into individual genre track listings.

**Contributors Tab** - Contributor profiles showing track counts, streak indicators (for consecutive year activity), top genres, niche genre specialization, and top artists. Includes filtering options to show tracks added by multiple contributors.

**Partners Tab** - Visualizes collaboration patterns between contributors by showing shared track additions. Displays Venn diagram-style badges and shared artist information.

**Playlists Tab** - Curated playlist browser with filtering by contributor and year. Displays contributor information, track counts, and direct Spotify integration.

## Technical Architecture

### Data Processing

The application implements a sophisticated multi-pass data analysis pipeline:

1. **CSV Parsing** - Uses PapaParse library to handle large CSV files with dynamic typing and skip-empty-lines optimization.

2. **Data Normalization** - All string data normalized through whitespace trimming and standardization to ensure consistent lookups across the dataset.

3. **Pre-computed Indexes** - During initialization, the system builds comprehensive lookup maps for O(1) access:
   - Track-to-genres mapping
   - Track-to-release dates
   - Track-to-contributors
   - Album-to-tracks (with deduplication)
   - Artist-to-tracks (with deduplication)
   - Genre-to-tracks (with deduplication)
   - Contributor statistics (years active, top artists, top genres, popularity metrics)

4. **Consolidated Track Arrays** - For frequently accessed detail views (artist details, genre details, album details), the system pre-computes deduplicated track arrays sorted by contributor count and recency. This eliminates runtime computation during navigation.

5. **Streaming Aggregations** - Year tracking, artist tracking, genre tracking, and contributor analysis are performed in a single pass through the raw data.

### Performance Optimizations

**Lazy Image Loading** - Album artwork uses the Intersection Observer API to load images only as they approach the viewport. Failed images are silently hidden without affecting layout.

**Deferred Rendering** - Detail page rendering (artist tracks, album tracks, genre tracks) is deferred with setTimeout to prevent UI blocking. Heavy computations execute in the next event loop cycle.

**Pagination** - Large result sets implement progressive loading with 20-item batches. Users can load additional tracks on demand rather than rendering thousands of cards at once.

**Lightweight Dependencies** - Uses only PapaParse for CSV parsing and Lucide for SVG icons. All other functionality implemented in vanilla JavaScript to minimize bundle size and parsing overhead.

**Memory Efficiency** - Sets are used for unique value tracking rather than arrays, reducing memory footprint. Contributors and shared artists are deduplicated at the source.

### Responsive Design

**Mobile-First CSS** - Media queries target specific breakpoints (480px, 768px, 900px, 1200px) with responsive grid layouts that adapt from single-column to multi-column based on screen width.

**Touch Interactions** - Touch-specific event handlers provide visual feedback on mobile devices. Cards include active/press states optimized for touch interfaces.

**Dynamic Menu** - Mobile screens replace horizontal tab navigation with a collapsible dropdown menu accessed via hamburger button.

**Optimized Tables** - On very small screens (480px), table views convert to card-based layouts with semantic restructuring of data for readability.

## User Interface

### Navigation

Desktop navigation uses a horizontal tab bar for primary views. Mobile devices implement a toggle-controlled dropdown menu. Back navigation button persists at bottom-right and maintains full navigation history.

### Scroll Position Preservation

The application maintains scroll positions for each tab view. When navigating away and returning, users are automatically scrolled to their previous position within that view.

### Search and Filtering

All views include contextual search boxes supporting real-time filtering by name or artist. Year-based filter buttons allow temporal slicing of data. Genre views support genre tag-based drilling. Contributor views include optional filtering for tracks added by multiple contributors.

### Visual Consistency

Cards use consistent padding (16px), border styling, and hover effects across all views. Accent color (#E3836D) provides visual continuity. Contributor badges use consistent color mapping across the application.

## Code Organization

The application structure consists of a single HTML file with embedded CSS and JavaScript for simplified deployment and no build process requirements.

### Key Functions

**analyzeData()** - Main data processing pipeline. Counts frequencies, builds lookup maps, pre-computes consolidated arrays, and initializes global data structures.

**renderOverview/renderArtists/renderTracks/renderAlbums/renderGenres/renderContributors/renderPartners/renderPlaylists()** - Tab content rendering functions. Each uses pre-computed global data to generate view-specific HTML.

**showArtistDetails/showAlbumDetails/showGenreDetails/showContributorDetails/showOverlapDetails()** - Detail view handlers. Implement pagination, lazy rendering, and scroll position management.

**generateTrackCardsHTML()** - Batch track card generation with genre tags, contributor badges, and Spotify linking. Used across multiple detail views.

**filterContributorCards/filterTrackCards/filterArtistCards/filterAlbumCards/filterGenreCards()** - Real-time search filtering using dataset.querySelectorAll() with case-insensitive matching.

### Global State

**globalData** - Central data store containing parsed raw data, pre-computed frequency counts, lookup maps, and consolidated arrays.

**contributorColors** - Persistent color mapping for contributor badges across all views.

**scrollPositions** - Tab-specific scroll position cache for preservation during navigation.

## Data Format

The system expects a CSV file with the following columns:

- Track ID (Spotify track identifier)
- Track Name
- Artist Name(s) (semicolon or ampersand delimited)
- Album Name
- Genre (semicolon delimited, supports " & " separators)
- Duration or Duration (ms)
- Release Date
- Added At (date added to collection)
- Added By (contributor identifier, expected format: firstname.lastname)
- Popularity (optional, numeric 0-100)
- Playlist Name (optional)
- Playlist URI (optional, Spotify URI format)

## Browser Compatibility

Tested and functional on Chrome, Firefox, Safari, and Edge. Requires support for ES6 features including arrow functions, template literals, Set/Map data structures, and const/let declarations.

## External Dependencies

- PapaParse (v5.3.0) - CSV parsing
- Lucide Icons (v0.263.1) - SVG icon library
- Spotify Web API - Track and playlist linking (read-only, no authentication required)

All dependencies loaded via CDN from cdnjs.cloudflare.com.

## Album Artwork

Track artwork is loaded from external GitHub repository. Expected URL format: `https://raw.githubusercontent.com/[organization]/[repository]/[branch]/[trackid].jpg`

Failed image loads are handled without affecting card layout or functionality.

https://github.com/mcmillon/music-week-album-art

## Known Limitations

- No persistent storage between sessions (data refreshes on page reload)
- Album artwork loading dependent on external image server availability
- Very large datasets (20,000+ unique tracks) may experience performance degradation on lower-end devices
- Contributors with many added tracks may show abbreviated name display for UI space constraints

## Development Notes

This application was developed with Claude, using iterative refinement to optimize performance and user experience. The data processing pipeline was specifically designed to handle datasets of 5,000+ tracks efficiently through pre-computation of frequently accessed aggregations.

The codebase prioritizes clarity and maintainability through consistent naming conventions and logical function organization. CSS uses CSS variables for theming, allowing future customization of the accent color and color scheme through root variable modification.
