# -
бот савка
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import requests

# Replace with your actual API key and endpoint
API_ENDPOINT = "https://api.example.com/historical_matches"  # Placeholder
API_KEY = "YOUR_API_KEY"  # Replace with your API key

def fetch_data(team, season):
    """Fetches historical match data from the API."""
    try:
        headers = {"X-API-Key": API_KEY}  # Some APIs require headers
        response = requests.get(f"{API_ENDPOINT}?team={team}&season={season}", headers=headers)
        response.raise_for_status()  # Raise HTTPError for bad responses
        data = response.json()
        return data
    except requests.exceptions.RequestException as e:
        st.error(f"Error fetching {e}")
        return None

def process_data(data):
    """Processes the raw data and returns a Pandas DataFrame."""
    if data is None:
        return pd.DataFrame()

    # This is a simplified example - adapt to your API's data structure
    matches = []
    for match in data:
        match_data = {
            "date": match["date"],
            "home_team": match["home_team"],
            "away_team": match["away_team"],
            "home_score": match["home_score"],
            "away_score": match["away_score"],
        }
        matches.append(match_data)
    return pd.DataFrame(matches)

def analyze_data(df):
    """Performs basic data analysis (example: win rate)."""
    if df.empty:
        st.warning("No data to analyze.")
        return

    # Example: Calculate home win rate
    df["home_win"] = df["home_score"] > df["away_score"]
    home_win_rate = df["home_win"].mean()
    st.write(f"Home Win Rate: {home_win_rate:.2f}")

    # Example: Calculate average total goals
    df["total_goals"] = df["home_score"] + df["away_score"]
    avg_goals = df["total_goals"].mean()
    st.write(f"Average Total Goals: {avg_goals:.2f}")

def visualize_data(df):
    """Creates visualizations (example: bar chart of goal distribution)."""
    if df.empty:
        st.warning("No data to visualize.")
        return

    # Example: Create a bar chart of goal distribution
    goal_counts = df["total_goals"].value_counts().sort_index()
    fig, ax = plt.subplots()
    ax.bar(goal_counts.index, goal_counts.values)
    ax.set_xlabel("Total Goals")
    ax.set_ylabel("Number of Matches")
    ax.set_title("Distribution of Total Goals")
    st.pyplot(fig)


def main():
    st.title("Sports Data Analyzer")

    # Sidebar inputs
    team = st.sidebar.text_input("Enter Team Name:", "Example Team")
    season = st.sidebar.text_input("Enter Season:", "2023")

    # Fetch and process data
    if st.button("Fetch and Analyze Data"):
        with st.spinner("Fetching data..."):
            raw_data = fetch_data(team, season)
            df = process_data(raw_data)

        if not df.empty:
            st.write("### Data Preview")
            st.dataframe(df.head()) # Show a preview of the data

            st.write("### Analysis")
            analyze_data(df)

            st.write("### Visualization")
            visualize_data(df)
        else:
            st.warning("No data found for the specified team and season.")

if __name__ == "__main__":
    main()
