pandas
matplotlib
seaborn
streamlit
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime, timedelta
from collections import defaultdict

# Function to detect schedule flaws
def detect_schedule_flaws(schedule_df):
    flaws = []
    
    # Check for overlapping tasks
    for i, row_i in schedule_df.iterrows():
        for j, row_j in schedule_df.iterrows():
            if i != j:
                overlap = max(0, min(row_i['End Date'], row_j['End Date']) - 
                             max(row_i['Start Date'], row_j['Start Date']))
                if overlap > 0:
                    flaws.append({
                        'type': 'Overlap',
                        'task1': row_i['Task ID'],
                        'task2': row_j['Task ID'],
                        'overlap_days': overlap.days
                    })
    
    # Check for negative durations
    negative_durations = schedule_df[schedule_df['Duration (days)'] <= 0]
    for _, row in negative_durations.iterrows():
        flaws.append({
            'type': 'Negative Duration',
            'task_id': row['Task ID'],
            'duration': row['Duration (days)']
        })
    
    return flaws

# Main app function
def main():
    st.title("Construction Schedule Detector")
    
    # File uploader
    uploaded_file = st.file_uploader("Upload your schedule file (CSV or Excel)", 
                                   type=['csv', 'xlsx', 'xls'])
    
    if uploaded_file is not None:
        try:
            # Read file
            if uploaded_file.name.endswith('.xlsx') or uploaded_file.name.endswith('.xls'):
                schedule_df = pd.read_excel(uploaded_file)
            else:
                schedule_df = pd.read_csv(uploaded_file)
            
            # Ensure date columns are properly formatted
            schedule_df['Start Date'] = pd.to_datetime(schedule_df['Start Date'])
            schedule_df['End Date'] = pd.to_datetime(schedule_df['End Date'])
            
            # Detect flaws
            flaws = detect_schedule_flaws(schedule_df)
            
            # Show results
            st.subheader("Detection Results")
            
            if not flaws:
                st.success("✅ No major flaws detected in your schedule!")
            else:
                st.warning("⚠ Flaws detected:")
                for flaw in flaws:
                    if flaw['type'] == 'Overlap':
                        st.write(f"- Tasks {flaw['task1']} and {flaw['task2']} overlap by {flaw['overlap_days']} days")
                    elif flaw['type'] == 'Negative Duration':
                        st.write(f"- Task {flaw['task_id']} has negative duration ({flaw['duration']} days)")
            
            # Show schedule visualization
            st.subheader("Schedule Timeline")
            fig, ax = plt.subplots(figsize=(12, 6))
            
            for _, row in schedule_df.iterrows():
                ax.barh(row['Task ID'], 
                       width=(row['End Date'] - row['Start Date']).days,
                       left=row['Start Date'],
                       height=0.5)
            
            ax.set_xlabel('Date')
            ax.set_ylabel('Tasks')
            ax.set_title('Construction Schedule Timeline')
            ax.grid(True, axis='x', linestyle='--', alpha=0.7)
            plt.tight_layout()
            st.pyplot(fig)
            
        except Exception as e:
            st.error(f"Error processing file: {str(e)}")

if _name_ == "_main_":
    main()
