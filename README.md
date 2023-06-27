
# Potential talents

## Background:

As a talent sourcing and management company, we are interested in finding talented individuals for sourcing these candidates to technology companies. Finding talented candidates is not easy, for several reasons. The first reason is one needs to understand what the role is very well to fill in that spot, this requires understanding the client’s needs and what they are looking for in a potential candidate. The second reason is one needs to understand what makes a candidate shine for the role we are in search for. Third, where to find talented individuals is another challenge.

The nature of our job requires a lot of human labor and is full of manual operations. Towards automating this process we want to build a better approach that could save us time and finally help us spot potential candidates that could fit the roles we are in search for. Moreover, going beyond that for a specific role we want to fill in we are interested in developing a machine learning powered pipeline that could spot talented individuals, and rank them based on their fitness.

We are right now semi-automatically sourcing a few candidates, therefore the sourcing part is not a concern at this time but we expect to first determine best matching candidates based on how fit these candidates are for a given role. We generally make these searches based on some keywords such as “full-stack software engineer”, “engineering manager” or “aspiring human resources” based on the role we are trying to fill in. These keywords might change, and you can expect that specific keywords will be provided to you.

Assuming that we were able to list and rank fitting candidates, we then employ a review procedure, as each candidate needs to be reviewed and then determined how good a fit they are through manual inspection. This procedure is done manually and at the end of this manual review, we might choose not the first fitting candidate in the list but maybe the 7th candidate in the list. If that happens, we are interested in being able to re-rank the previous list based on this information. This supervisory signal is going to be supplied by starring the 7th candidate in the list. Starring one candidate actually sets this candidate as an ideal candidate for the given role. Then, we expect the list to be re-ranked each time a candidate is starred.

## Data Description:

The data comes from our sourcing efforts. We removed any field that could directly reveal personal details and gave a unique identifier for each candidate.

## Attributes:
id : unique identifier for candidate (numeric)

job_title : job title for candidate (text)

location : geographical location for candidate (text)

connections: number of connections candidate has, 500+ means over 500 (text)

Output (desired target):
fit - how fit the candidate is for the role? (numeric, probability between 0-1)

Keywords: “Aspiring human resources” or “seeking human resources”

## Download Data:

https://docs.google.com/spreadsheets/d/117X6i53dKiO7w6kuA1g1TpdTlv1173h_dPlJt5cNNMU/edit?usp=sharing

## Goal(s):

Predict how fit the candidate is based on their available information (variable fit)

## Success Metric(s):

Rank candidates based on a fitness score.

Re-rank candidates when a candidate is starred.
# Method and Results
## Exploratory data analysis
I first explored the data to gain insights. This included looking at how many entries we have (104) and how many features (3, after removing an empty column).

I decided to add 3 negative control entries in my data so I could check the accuracy of the similarity code I use later on.

After so more in depth exploration of the data I saw there were duplicate entries, which I removed, leaving us with 56 entries (including 3 controls)

## Cleaning data 
There were a lot of acronyms in the job titles, which I replaced. The punctuaction was removed, and everything was put into lowercase.
I also normalized the connection column to be between 0-1.

## Word embedding
I calculated the similarity between the job titles and the given keyword (stored as variable 'keywords' in the top of the code, at first it we use 'aspiring human resources')
I first tried word2vec, then BERT to encode the job titles, then used cosine similarity to compute thesimilarity to the variable 'keywords'.
My control entries show up quite low when sorting by word2vec similarity, better than with BERT. This is likely because word2vec is a keyword based model that works better in this usecase.
I then defined a fitness score based on an equation taking similarity_score (word2vec) and normalized_connections into account. Sorting by this fitness score gives me an initial ranking.

## Starring candidates
This exercise assumes a human in the loop at this stage where an HR expert would star candidates that seem promising. To emulate this I selected 1 high ranking candidate to mark as starred. I then redefined my fitness function to account for the starred statyus and reranked the data accordingly.
I then starred 3 more high ranking candidates as an example.

## Ranking model

I then built a ranking model using LightGBM. I used the predictions from this ranking model to update the ranking in the database.
The previously starred candidates appear near the top, suggesting the model learned how to rank somewhat.
I chose to star 7 more candidates and retrain the model to see if it ranked better.
The results now show better results concerning the placement of our starred candidates in the ranking, suggesting good model training.


## Bonus questions

### We are interested in a robust algorithm, tell us how your solution works and show us how your ranking gets better with each starring action 
- We can see the ranking is improved as we increase the starred candidate sample size

### How can we filter out candidates which in the first place should not be in this list? 
- We can filter candidates based on their word2vec similarity score, or based on the fitness column. The max/min for those (before starring) is 0.87/0.04 and 0.79/0.04 respectively. We can establish a threshold based on that and automatically eliminate candidates below a certain threshold. There is sudden dip in similarity scores from 0.51 to 0.35 so perhaps 0.5 can be considered the threshold


### Do you have any ideas that we should explore so that we can even automate this procedure to prevent human bias?
- To prevent human bias we could automate the model and addother columns of candidate information such as years of experience or skillset keywords, which could help make the model more accurate
