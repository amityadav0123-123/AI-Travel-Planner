# AI-Travel-Planner

%%writefile app.py

import os
import streamlit as st
from crewai import Agent, Task, Crew, Process, LLM
from crewai_tools import SerperDevTool



# Initialize search tool for real-time travel info
search_tool = SerperDevTool()

# Define AI Model
llm = LLM(model="gemini/gemini-1.5-flash",
          verbose=True,
          temperature=0.5,
          api_key=os.environ["GOOGLE_API_KEY"])

# Define function to create agents
def create_agents():
    researcher = Agent(
        role="Travel Researcher",
        goal=f"Find historical sites, public transport hotels, and real-time weather for {destination}.",
        backstory="You are an expert travel researcher, providing up-to-date information about history-focused trips.",
        verbose=True,
        memory=True,
        llm=llm,
        tools=[search_tool]
    )

    budget_planner = Agent(
        role="Budget Planner",
        goal=f"Find budget flights, hotels, and activities within {budget} for {destination}.",
        backstory="You are a skilled budget analyst ensuring trips fit within financial constraints.",
        verbose=True,
        memory=True,
        llm=llm,
        tools=[search_tool]
    )

    itinerary_planner = Agent(
        role="Itinerary Planner",
        goal=f"Create a 3-day itinerary for {destination}, ensuring all historical sites are covered under {budget}.",
        backstory="You are an expert in trip planning, ensuring travelers get the best experience within their budget.",
        verbose=True,
        memory=True,
        llm=llm,
        tools=[search_tool]
    )

    return researcher, budget_planner, itinerary_planner

# Streamlit UI
st.title("🌍 AI-Powered Travel Planner")

destination = st.text_input("Enter Destination:")
budget = st.text_input("Enter Budget (INR):")

if st.button("Generate Travel Plan"):
    if not destination or not budget:
        st.error("Please enter both a destination and budget.")
    else:
        st.info("🧳 Generating your travel plan...")

        # Create agents
        researcher, budget_planner, itinerary_planner = create_agents()

        # Define tasks
        research_task = Task(
            description=f"Find the best historical sites, weather forecast, and public transport hotels for {destination}.",
            expected_output="A list of top historical sites, a real-time weather update, and 3 hotel options near public transport.",
            agent=researcher
        )

        budget_task = Task(
            description=f"Find budget flights, hotel options, and daily costs for {destination} within {budget}.",
            expected_output=f"A full cost breakdown (flights, hotel, food, attractions) ensuring a {budget} budget is maintained.",
            agent=budget_planner
        )

        itinerary_task = Task(
            description=f"Plan a 3-day itinerary for {destination} under {budget}.",
            expected_output="A detailed 3-day plan, considering weather and budget constraints, with transport recommendations.",
            agent=itinerary_planner
        )

        # Create Crew
        crew = Crew(
            agents=[researcher, budget_planner, itinerary_planner],
            tasks=[research_task, budget_task, itinerary_task],
            process=Process.sequential
        )

        # Run AI agents
        responses = crew.kickoff(inputs={'destination': destination, 'budget': budget})

         # Display each agent's output separately
        st.subheader("📌 Travel Researcher Findings")
        st.write(responses[0])

        st.subheader("💰 Budget Planner Suggestions")
        st.write(responses[1])

        st.subheader("🗺️ Itinerary Planner Recommendations")
        st.write(responses[2])

        # Display results
        st.success("✅ Travel Plan Generated!")
        st.subheader("Your Travel Plan:")
        st.write(responses)
