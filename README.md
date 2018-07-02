# Automatic Playlist Continuation using SubProfile-Aware Diversification

This repository contains the source code to reproduce the results obtained by our team "teamrozik" in the Spotify RecSys Challenge 2018, Creative Track. 

## Team Info
Team Name: teamrozik
Challenge Track: creative

## Dependencies
The main python module dependencies are:
- numpy
- pandas
- operator
- json

**Note**: We have used Python 2.7.12 :: Anaconda 4.1.1 (64-bit). For Python 3.x, it is still usable with minor modification. Some of our python module codes are adapted from "[vae_cf](https://github.com/dawenl/vae_cf)", for preprocessing the data, and also we use some of the code that is released together with MPD dataset. 

The main Java module dependencies can be found in the pom.xml file. Our implementation is based on "[RankSYS Framework](https://github.com/RankSys/RankSys)". src/nn folder is from the Github repository of RankSYS, since in the maven repositories the version of RankSys is 0.4.3 but on GitHub it is 0.4.4-snapshot and our implementation is based on the Github version. The code in src/mf is also from RankSys, we only added a random seed for the initialization of the latent factors of Matrix Factorization algorithm for reproducability. 

**Note**: We have used Java(TM) SE Runtime Environment (build 1.8.0_101-b13) in our experiments.    


## Approach

We submit to the Creative Track, because we use known tracks of the Challenge Set playlists to train our model. Using challenge set is considered as external data as well. 

We split this problem into two:

### Playlist with Cold-start scenario

For the cold-start scenario, we consider 1000 playlists that has title only and 1000 playlists with title and first track. 

For the playlists with title info only, we create a title popularity recommender, which basically recommends the tracks that are in the playlists in MPD having the same title. Details can be found in the title_popularity_recommendations.py file. 

For the playlists with title and first track only again we create a popularity recommender, but this time we consider not only title but also the playlists having the track as well. Details can be found in the title_one_song_popularity_recommendations.py file. 

We also run a popularity recommender, because for some cold-start playlists our approaches may not produce 500 recommendations.
 
**Note** We were planning to look for semantic similarity of the titles and string similarity of the titles, but we did not have anough time to test these ideas. 
### Other Playlists

Our idea is based on our recent publication "[Accurate and Diverse Recommendations Using Item-Based SubProfiles](https://aaai.org/ocs/index.php/FLAIRS/FLAIRS18/paper/view/17600)", which detect subprofiles of the users that corresponds to different interests or tastes of the user and uses the detected subprofiles to diversify the recommendations generated by a baseline recommender (Matrix Factorization for example).

Our assumption is that in the user generated playlists as well, there are sub-profiles that corresponds to user's different interests or tastes. Our goal is to cover those subprofiles of the playlists while producing the final set of recommendations. 
 
For the rest of 8000 playlists we consider each playlist as a user and tracks as items to be used in a Matrix Factorization recommender. As we explain later, by using preprocess.py we get <playlistID,trackID,title> fro the playlists in MPD and 8000 playlist in the challenge set. We filter the tracks appearing in 1 playlist only and map the playlist and trackIDs. We use "[Fast ALS-based factorization of Pilászy, Zibriczky and Tikk.](https://dl.acm.org/citation.cfm?id=1864726)". We use RankSys implementation.  

After generating the baseline recommendations, we re-rank the recommendations for those 8000 playlsits based on their detected subprofiles. 

**Note** Our approach to detecting the subprofiles is not the same as our recent publication, it is a simpler and more effective approach that we use for this challenge. Details are in the SubProfileExtraction.java file. 

## Producing recommendations

The python codes are under playlist_challenge folder. First thing to do is to change MPD_PATH  to the correct path in your environment.
For filtering the dataset and preprocessing run the following command:
```
python preprocess.py 
```

After running the above script, run:

```
python title_popularity_recommendations.py
```

Then run:

```
python title_one_song_popularity_recommendations.py
```

The Java codes for our SubProfile-Aware Diversification (SPAD) are under SpotifyChallenge/src. First, from the training data, we pre-compute track-track similarities by using src/spotify_challenge/PreComputeItemSims.java. Change MPD_PATH with the path of your environment.

Then for detecting the subprofiles of the 8000 playlists in the challenge set use src/spotify_challenge/SubProfileExtraction.java. Do not forget to change MPD_PATH.

Then run src/spotify_challenge/MFRecommenderExample.java to produce 500 recommendations for 8000 playlists in the challenge set. 

Re-rank recommendations produced in the previous step by using src/spotify_challenge/SPADReRanker.java. 

Run src/spotify_challenge/PopularityRecommenderExample.java to produce 500 popular recommendations that will be used in cold start playlists. 

**Note** We optimized the hyper-parameters of the MAtrix Factorization algorithm and SPAD by using 10000 random playlists from MPD as validation set. We split data of those playlists as 80% train and 20%validation. Optimized hyper-parameters are the one we use in the codes in this repository.


## Compiling by Maven and running the code!
As explained above in all the files under spotify_challenge package change MPD_PATH. Then the classes can be compiled and run as follows: 

```
JAVA_HOME=/usr/java/default/ mvn clean compile
export MAVEN_OPTS="-Xmx24000M"
mvn exec:java -Dexec.mainClass="spotify_challenge.PreComputeItemSims"
mvn exec:java -Dexec.mainClass="spotify_challenge.SubProfileExtraction"
mvn exec:java -Dexec.mainClass="spotify_challenge.MFRecommenderExample"
mvn exec:java -Dexec.mainClass="spotify_challenge.SPADReRanker"
mvn exec:java -Dexec.mainClass="spotify_challenge.PopularityRecommenderExample"
```

**NOTE** Do not forget to replace jdk path instead of /usr/java/default above. 
## Creating submissions.

Finally in the python module run:
```
python create_submission.py
```


