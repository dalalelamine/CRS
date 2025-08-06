
# Data Preparation Documentation

## Overview

This document describes the comprehensive data preparation pipeline developed for creating a two-tower recommendation model dataset. The pipeline combines MovieLens 25M ratings data with The Movie Database (TMDB) metadata to create enriched user and item representations suitable for neural collaborative filtering approaches.

## Data Sources

### Primary Dataset: MovieLens 25M
- **Source**: Kaggle (veeralakrishna/movielens-25m-dataset)
- **Size**: 25 million ratings from 162,541 users on 62,423 movies
- **Core Files**:
  - `movies.csv`: Movie metadata with titles and genres
  - `ratings.csv`: User-movie ratings (1-5 scale) with timestamps
  - `links.csv`: Mapping between MovieLens IDs and external databases (TMDB, IMDB)

### Secondary Dataset: The Movie Database (TMDB)
- **Source**: TMDB API v3
- **Purpose**: Enhanced movie metadata and cast/crew information
- **Access**: RESTful API with rate limiting (40 requests/10 seconds)

## Data Processing Pipeline

### 1. Data Acquisition and Loading (`load_movielens_data`)

The pipeline begins by automatically downloading and extracting the MovieLens dataset from Kaggle. The system implements robust file discovery to locate CSV files within the extracted archive structure.

**Key Features**:
- Automatic download and extraction with error handling
- Flexible file path discovery for different archive structures
- Validation of required files (movies.csv, ratings.csv, links.csv)

### 2. TMDB Data Enrichment (`fetch_tmdb_data_batch`)

This phase enriches the MovieLens movies with comprehensive metadata from TMDB, including detailed movie information and cast/crew data.

**API Integration Features**:
- Batch processing with configurable batch sizes (default: 100 movies)
- Exponential backoff retry mechanism for failed requests
- Rate limiting compliance (250ms delay between requests)
- Robust error handling for network timeouts and API errors

**Retrieved TMDB Attributes**:
- **Basic Metadata**: Title, overview, release date, runtime, popularity scores
- **Content Ratings**: Vote average, vote count, adult content flags
- **Visual Assets**: Poster and backdrop image paths
- **Categorical Data**: Genres, production countries, original language
- **Cast and Crew**: Top 5 cast members and director information

### 3. Data Integration and Merging (`merge_movielens_tmdb`)

The integration phase combines MovieLens data with TMDB metadata through a series of left joins:

1. **MovieLens-Links Join**: Connects movies with external database IDs
2. **TMDB Integration**: Merges enhanced metadata using TMDB IDs as keys
3. **Data Validation**: Ensures referential integrity across merged datasets

### 4. User Behavior Analysis and Preference Extraction

#### User Preference Mining (`create_user_preferences`)
The system analyzes user rating patterns to extract implicit preferences:

- **High Rating Threshold**: Filters ratings ≥ 4.0 as positive preferences
- **Genre Preference Ranking**: Counts genre frequencies in highly-rated movies
- **Top-K Selection**: Extracts top 3 preferred genres per user using frequency analysis

#### Synthetic Demographics Generation (`synthesize_user_demographics`)
To enable demographic-aware recommendations, the pipeline generates realistic synthetic user profiles:

**Demographic Attributes**:
- **Geographic Distribution**: 8 countries with representative city distributions
- **Age Demographics**: Birth years spanning 1960-2005 (ages 20-65)
- **Age Categorization**: Six-tier system (Child, Teenager, Young, Mid Age, Old, Senior)

**Synthetic Data Distribution**:
```
Countries: USA, Canada, UK, India, Germany, France, Australia, Japan
Age Categories: Child (<13), Teenager (13-17), Young (18-29), 
                Mid Age (30-49), Old (50-64), Senior (65-100)
```

### 5. Data Cleaning and Quality Assurance

#### Movie Data Cleaning (`clean_movie_data`)
- **Completeness Filtering**: Removes movies lacking TMDB metadata
- **Column Optimization**: Drops redundant columns (e.g., imdbId)
- **Data Type Validation**: Ensures appropriate data types for all attributes

#### Ratings Data Processing (`process_ratings_data`)
- **Timestamp Removal**: Simplifies temporal aspects for static modeling
- **Genre Integration**: Temporarily merges genres for preference analysis
- **Final Cleanup**: Removes intermediate columns after preference extraction

## Final Dataset Schema

### Movies Dataset
| Column | Type | Description |
|--------|------|-------------|
| movieId | int | MovieLens unique identifier |
| title | str | Original MovieLens title |
| genres | str | Pipe-separated genre list |
| tmdbId | int | TMDB unique identifier |
| tmdb_title | str | TMDB canonical title |
| overview | str | Movie plot summary |
| popularity | float | TMDB popularity score |
| release_date | str | Release date (YYYY-MM-DD) |
| runtime | int | Duration in minutes |
| vote_average | float | TMDB average rating |
| vote_count | int | Number of TMDB votes |
| tmdb_genres | str | TMDB genre categories |
| production_countries | str | Countries of origin |
| top_cast | str | Top 5 cast members |
| director | str | Primary director |

### Users Dataset
| Column | Type | Description |
|--------|------|-------------|
| userId | int | Unique user identifier |
| preferred_genres | str | Top 3 preferred genres |
| birthdate | str | Synthetic birth date |
| nationality | str | Synthetic nationality |
| location | str | Synthetic city location |
| age | int | Calculated current age |
| age_category | str | Age group classification |

### Ratings Dataset
| Column | Type | Description |
|--------|------|-------------|
| userId | int | User identifier |
| movieId | int | Movie identifier |
| rating | float | Rating score (1.0-5.0) |

## Quality Metrics and Statistics

### Data Coverage and Completeness
- **TMDB Match Rate**: Percentage of MovieLens movies successfully enriched
- **User Coverage**: Users with sufficient rating history for preference extraction
- **Rating Distribution**: Ensures balanced representation across rating scales

### Synthetic Data Validation
- **Geographic Distribution**: Realistic population-weighted country selection
- **Age Distribution**: Normal distribution centered on active movie-viewing demographics
- **Preference Consistency**: Genre preferences align with actual rating patterns

## Technical Implementation Details

### Performance Optimizations
- **Batch Processing**: Reduces API overhead through batched TMDB requests
- **Memory Management**: Processes data in chunks to handle large datasets
- **Caching Strategy**: Implements local file caching to avoid redundant downloads

### Error Handling and Robustness
- **Network Resilience**: Implements retry mechanisms with exponential backoff
- **API Rate Limiting**: Respects TMDB API constraints with appropriate delays
- **Data Validation**: Comprehensive input validation and error reporting

### Reproducibility Features
- **Random Seed Control**: Fixed seed (42) for consistent synthetic data generation
- **Configuration Management**: Centralized parameter settings for easy modification
- **Logging and Monitoring**: Detailed progress reporting and error tracking

## Usage Instructions

### Prerequisites
- Python 3.7+ with pandas, requests, and standard libraries
- Valid TMDB API key (free registration required)
- Stable internet connection for data downloads

### Execution Options
1. **Sample Mode**: Process 100 movies for testing and validation
2. **Full Dataset**: Complete 25M rating dataset (requires several hours)

### Output Files
- `prepared_movies.csv`: Enhanced movie metadata
- `prepared_users.csv`: User profiles with preferences and demographics
- `prepared_ratings.csv`: Clean user-movie ratings

## Limitations and Considerations

### API Dependencies
- **Rate Limiting**: TMDB API constraints limit processing speed
- **Data Availability**: Not all MovieLens movies have TMDB matches
- **API Changes**: Future TMDB API modifications may require code updates

### Synthetic Data Limitations
- **Demographic Realism**: Synthetic profiles may not reflect true user distributions
- **Privacy Considerations**: Real demographic data unavailable due to privacy constraints
- **Bias Potential**: Synthetic generation may introduce unintended biases

### Scalability Constraints
- **Memory Requirements**: Full dataset processing requires substantial RAM
- **Processing Time**: Complete pipeline execution takes several hours
- **Storage Needs**: Final datasets require significant disk space

## Future Enhancements

### Data Enrichment Opportunities
- **Additional APIs**: Integration with other movie databases (OMDB, Rotten Tomatoes)
- **Real-time Updates**: Periodic refresh of movie metadata and ratings
- **Enhanced Demographics**: More sophisticated synthetic profile generation

### Performance Improvements
- **Parallel Processing**: Multi-threaded API requests within rate limits
- **Database Integration**: Direct database storage for improved query performance
- **Incremental Updates**: Delta processing for new ratings and movies

# Two-Tower Movie Recommendation System - Methodology

## Overview

This methodology presents a comprehensive two-tower neural architecture for movie recommendation systems, implemented using TensorFlow and Keras. The system employs separate neural networks (towers) to learn embeddings for users and movies independently, which are then combined to predict user preferences and generate personalized recommendations.

## Architecture Design

### Two-Tower Framework

The recommendation system is built upon a two-tower architecture consisting of three main components:

1. **User Tower**: Processes user-specific features to generate user embeddings
2. **Movie Tower**: Processes movie-specific features to generate movie embeddings  
3. **Interaction Layer**: Combines user and movie embeddings to compute similarity scores

This architecture enables efficient candidate retrieval and personalized ranking by learning dense representations in a shared embedding space.

### Model Components

#### User Tower Architecture

The user tower (`create_user_tower`) processes multiple categorical and textual user features:

**Input Features:**
- `watched_titles`: Textual representation of previously watched movies
- `preferred_genres`: User's preferred movie genres
- `age_category`: Categorical age group
- `nationality`: User's nationality
- `location`: Geographic location

**Processing Pipeline:**
1. **Text Vectorization**: Watched movie titles are processed using `TextVectorization` with a vocabulary of 10,000 tokens and sequence length of 20
2. **Categorical Encoding**: Categorical features are encoded using `StringLookup` layers
3. **Embedding Layers**: Each feature type is mapped to dense embeddings:
   - Watched titles: 64-dimensional embeddings with Global Average Pooling
   - Genres: 8-dimensional embeddings
   - Age category: 4-dimensional embeddings
   - Nationality: 8-dimensional embeddings
   - Location: 8-dimensional embeddings
4. **Feature Fusion**: All embeddings are concatenated and passed through dense layers (128 → 64 units) with ReLU activation

#### Movie Tower Architecture

The movie tower (`create_movie_tower`) processes rich movie metadata:

**Input Features:**
- `title`: Movie title
- `genres`: Movie genres
- `overview`: Plot synopsis/description
- `top_cast`: Main cast members
- `director`: Director information

**Processing Pipeline:**
1. **Text Vectorization**: Each textual feature uses specialized vectorizers:
   - Title: 10,000 vocab, 20 sequence length, 64-dim embeddings
   - Genres: 5,000 vocab, 10 sequence length, 32-dim embeddings
   - Overview: 15,000 vocab, 40 sequence length, 64-dim embeddings
   - Cast: 10,000 vocab, 20 sequence length, 64-dim embeddings
   - Director: 5,000 vocab, 5 sequence length, 32-dim embeddings
2. **Global Average Pooling**: Applied to each embedding sequence
3. **Feature Concatenation**: All movie embeddings are concatenated
4. **Dense Layers**: Final processing through 128 → 64 unit dense layers with ReLU activation

#### Two-Tower Integration

The `ImprovedMovieLensTwoTowerModel` combines the user and movie towers:

1. **Embedding Generation**: Both towers generate 64-dimensional embeddings
2. **Similarity Computation**: Two similarity metrics are supported:
   - **Dot Product**: Direct element-wise multiplication and summation
   - **Cosine Similarity**: L2-normalized embeddings followed by dot product
3. **Output**: Single similarity score representing user-movie preference

## Data Processing and Training

### Training Data Preparation

The `prepare_training_data` method creates training instances by:

1. **Rating Integration**: Merging user ratings with movie features
2. **Bias Correction**: Computing unbiased scores by subtracting user average ratings
3. **User History**: Aggregating watched movie titles for users with ratings ≥ 4.0
4. **Profile Integration**: Incorporating user demographic and preference data
5. **Missing Value Handling**: Filling missing categorical values with "Unknown"

### Training Procedure

The system supports both regression and classification training modes:

**Regression Mode:**
- Loss Function: Mean Squared Error (MSE)
- Metrics: Mean Absolute Error (MAE)
- Target: Continuous rating scores

**Classification Mode:**
- Loss Function: Binary Cross-Entropy
- Metrics: Binary Accuracy
- Target: Binary interaction indicators

**Training Features:**
- Batch processing with configurable batch sizes (default: 128)
- Adam optimizer with configurable learning rates
- Validation monitoring for overfitting prevention
- Checkpoint saving every 3 epochs for model persistence
- Comprehensive logging for training monitoring

## Inference and Recommendation Generation

### Recommendation Pipeline

The `get_movie_recommendations` method generates personalized recommendations through:

1. **User Profile Retrieval**: Extracting user features and viewing history
2. **Candidate Scoring**: Computing similarity scores for all candidate movies
3. **Batch Processing**: Efficient processing of large movie catalogs (batch size: 500)
4. **Filtering**: Removing previously rated movies from recommendations
5. **Ranking**: Sorting candidates by predicted preference scores

### API Interface

The system provides a streamlined API (`get_movie_recommendations_api`) for real-time inference:
- Accepts user profile dictionaries
- Processes movie catalogs in batches
- Returns top-N recommendations with scores

## Model Persistence and Serialization

### Saving Mechanism

The system implements comprehensive model serialization:

1. **Component Separation**: User tower, movie tower, and two-tower models saved independently
2. **Metadata Storage**: Configuration and versioning information in JSON format
3. **Auxiliary Data**: User viewing history stored as CSV files
4. **Keras Compatibility**: All models use native Keras serialization for reliability

### Loading and Restoration

Model restoration ensures complete system recovery:
- Validates all required components exist
- Restores individual tower models
- Re-establishes two-tower model references
- Loads auxiliary data and metadata

## Technical Implementation Details

### Framework and Dependencies

- **TensorFlow 2.x**: Primary deep learning framework
- **Keras**: High-level neural network API
- **Pandas**: Data manipulation and analysis
- **NumPy**: Numerical computations

### Memory and Performance Optimizations

1. **Batch Processing**: Large datasets processed in configurable batches
2. **Prefetching**: TensorFlow data pipeline optimizations with `tf.data.AUTOTUNE`
3. **Embedding Dimensions**: Carefully chosen dimensions balancing expressiveness and efficiency
4. **Text Processing**: Efficient vectorization with vocabulary limits and sequence truncation

### Error Handling and Validation

- Input validation for missing user profiles
- Fallback mechanisms for unknown users
- Model state verification before inference
- Comprehensive exception handling throughout the pipeline

## Evaluation and Monitoring

### Training Monitoring

- Real-time loss and accuracy tracking
- Validation metrics for generalization assessment
- Epoch-by-epoch logging with timestamps
- Model checkpoint creation for recovery

### Recommendation Quality

The system supports evaluation through:
- Similarity score distributions
- Recommendation diversity analysis
- User coverage metrics
- Computational efficiency measurements

This two-tower architecture provides a scalable, maintainable solution for large-scale movie recommendation systems, combining the flexibility of deep learning with the efficiency requirements of production recommendation systems.