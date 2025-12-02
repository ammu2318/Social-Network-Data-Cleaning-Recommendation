📘 README — Social Network Data Cleaning & Recommendation System
📝 Project Overview

This project processes a small social-network dataset stored in JSON format.
It contains:

Users (id, name, friends, liked_pages)

Pages (id, name)

We perform:

Data Cleaning

Friend Recommendations ("People You May Know")

Page Recommendations ("Pages You May Like")

The goal is to clean the dataset and build simple algorithms that suggest new friends and new pages for each user.

📂 Dataset Structure
Users
{
  "id": 1,
  "name": "Amit",
  "friends": [2, 3],
  "liked_pages": [101]
}

Pages
{
  "id": 101,
  "name": "Python Developers"
}

Issues found in raw data:

Some users have missing/blank names

Some users have duplicate friends

Some users have no friends (inactive users)

Pages have duplicate IDs (same page id repeated)

Friend IDs sometimes refer to non-existing users

Pages may also be duplicated or mismatched

🧹 Data Cleaning Implemented

We wrote a function clean_data() to clean the dataset before running recommendations.

✔ 1. Remove users with missing names
data["users"] = [user for user in data["users"] if user["name"].strip()]

✔ 2. Remove duplicate friends
for user in data["users"]:
    user["friends"] = list(set(user["friends"]))

✔ 3. Remove inactive users (users with no friends)
data["users"] = [user for user in data["users"] if user["friends"]]

✔ 4. Remove duplicate pages

We use a dictionary because it cannot have duplicate keys:

unique_pages = {}
for page in data["pages"]:
    unique_pages[page["id"]] = page
data["pages"] = list(unique_pages.values())


This ensures each page ID appears only once.

👥 People You May Know — Friend Recommendation

This algorithm recommends people a user may know based on mutual friends.

Logic:

Build a map: user_id → set(friends)

If the target user does not exist → return empty list

Get all direct friends of the user

Look at each friend’s friend list (friends of friends)

Count how many times each candidate appears

Exclude:

The user themself

Existing direct friends

Sort by mutual friend count (higher → better)

Returns:

A list of suggested user IDs ordered by score.

Example:

For user 1:

Friends: {2, 3}

Friend 2’s friends: {1, 4}

Friend 3’s friends: {1}

User 4 appears once → suggested.

Output:

[4]

📄 Friend Recommendation Code
def find_people_you_may_know(user_id, data):
    user_friends = {user["id"]: set(user["friends"]) for user in data["users"]}

    if user_id not in user_friends:
        return []

    direct_friends = user_friends[user_id]
    suggestions = {}

    for friend in direct_friends:
        for mutual in user_friends.get(friend, set()):
            if mutual != user_id and mutual not in direct_friends:
                suggestions[mutual] = suggestions.get(mutual, 0) + 1

    sorted_suggestion = sorted(suggestions.items(), key=lambda x: x[1], reverse=True)
    return [user_id for user_id, _ in sorted_suggestion]

📘 Pages You May Like — Page Recommendation

This algorithm recommends new pages to a user based on:

How many pages they share with other users

What other users like that the target user does not like

Logic:

Build a map: user_id → set(liked_pages)

For each other user:

Find intersection of liked pages

Score pages using number of shared pages

Add score for each suggested page

Sort by score (descending)

Return only the page IDs

📄 Page Recommendation Code
def recommend_pages(user_id, data):
    user_pages = {user['id']: set(user['liked_pages']) for user in data['users']}

    if user_id not in user_pages:
        return []

    user_liked_pages = user_pages[user_id]
    page_suggestion = {}

    for other_user, pages in user_pages.items():
        if other_user != user_id:
            shared_pages = user_liked_pages.intersection(pages)
            for page in pages:
                if page not in user_liked_pages:
                    page_suggestion[page] = page_suggestion.get(page, 0) + len(shared_pages)

    sorted_pages = sorted(page_suggestion.items(), key=lambda x: x[1], reverse=True)
    return [page_id for page_id, _ in sorted_pages]

🧪 Example Dry Run for Page Recommendations

User 1 likes: {101}
User 3 likes: {101, 103}
Shared pages = {101} → score = 1 for page 103.
User 4 likes 104, but no shared pages → score stays 0.

Final suggested pages:

[103]

🧠 Key Python Concepts Used

List comprehensions

Set operations (intersection, membership)

Dictionary .get() for safe access

Sorting with lambda

Tuple unpacking (for page_id, _ in sorted_list)

Deduplication using dictionary keys

JSON file handling (json.load, json.dump)

🚀 What This Project Demonstrates

✓ Data cleaning techniques
✓ Recommendation algorithms
✓ Processing JSON datasets
✓ Python intermediate-level skills
✓ Practical real-world data transformations

📌 Next Possible Improvements

Add explanation of why a user was recommended

Weight suggestions by popularity

Remove invalid friend references

Visualize friend graphs using NetworkX

Build a small Flask or Streamlit UI
