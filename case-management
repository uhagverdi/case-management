import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
import sqlite3
from sqlalchemy import create_engine

# 🏛 Database Setup
DB_FILE = "case_management.db"
engine = create_engine(f"sqlite:///{DB_FILE}")
conn = engine.connect()

# 📌 Step 1: Generate Dummy Case Data (For Initial Testing)
def generate_dummy_cases():
    data = {
        "Case_ID": [f"C{i}" for i in range(1, 101)],
        "Trader_ID": np.random.randint(1000, 2000, 100),
        "Case_Type": np.random.choice(["Insider Trading", "Wash Trading", "Sanctions Violation", "AML Violation"], 100),
        "Risk_Score": np.random.randint(10, 100, 100),
        "Assigned_To": np.random.choice(["Analyst A", "Analyst B", "Unassigned"], 100),
        "Status": np.random.choice(["Open", "In Progress", "Closed"], 100, p=[0.5, 0.3, 0.2]),
        "Date_Flagged": pd.date_range(start="2025-01-01", periods=100, freq="D").astype(str),
        "Comments": ["-" for _ in range(100)]
    }
    df = pd.DataFrame(data)
    df.to_sql("cases", engine, if_exists="replace", index=False)
    return df

# 📌 Step 2: Load Case Data
def load_cases():
    try:
        return pd.read_sql("SELECT * FROM cases", conn)
    except Exception:
        return generate_dummy_cases()

# 📊 Streamlit App Configuration
st.set_page_config(page_title="📌 Compliance Case Management", layout="wide")
st.title("🔍 Compliance Case Management System")

# 📂 Load Cases
df = load_cases()

# 🎯 **Case Filters**
st.sidebar.header("🔍 Filter Cases")
status_filter = st.sidebar.multiselect("Select Status", df["Status"].unique(), default=df["Status"].unique())
risk_filter = st.sidebar.slider("Minimum Risk Score", min_value=0, max_value=100, value=20)
case_type_filter = st.sidebar.multiselect("Select Case Type", df["Case_Type"].unique(), default=df["Case_Type"].unique())

# 📍 Apply Filters
df_filtered = df[(df["Status"].isin(status_filter)) & 
                 (df["Risk_Score"] >= risk_filter) & 
                 (df["Case_Type"].isin(case_type_filter))]

# 📌 **Case Overview**
st.subheader("📋 Case Overview Table")
st.dataframe(df_filtered)

# 📊 **Risk Score Distribution**
st.subheader("📈 Risk Score Distribution")
fig_risk = px.histogram(df_filtered, x="Risk_Score", nbins=20, color="Case_Type")
st.plotly_chart(fig_risk, use_container_width=True)

# 📊 **Case Status Breakdown**
st.subheader("📊 Case Status Breakdown")
fig_status = px.pie(df_filtered, names="Status", title="Case Status Breakdown")
st.plotly_chart(fig_status, use_container_width=True)

# 🎯 **Update Case Status**
st.subheader("✏️ Manage Cases")
case_selection = st.selectbox("Select a Case ID to Update", df_filtered["Case_ID"].unique())

new_status = st.selectbox("Update Status", ["Open", "In Progress", "Closed"])
new_comments = st.text_area("Add Comments (Optional)")

if st.button("✅ Update Case"):
    df.loc[df["Case_ID"] == case_selection, "Status"] = new_status
    df.loc[df["Case_ID"] == case_selection, "Comments"] = new_comments
    df.to_sql("cases", engine, if_exists="replace", index=False)
    st.success(f"🔄 Case {case_selection} updated successfully!")

# 📌 **Case Reports**
st.subheader("📄 Generate Compliance Reports")
if st.button("📂 Download Case Report"):
    df_filtered.to_csv("compliance_case_report.csv", index=False)
    st.success("✅ Report generated! Check your downloads folder.")
