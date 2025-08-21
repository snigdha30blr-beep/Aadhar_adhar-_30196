# Aadhar_adhar-_30196
Code for the frontend, backend, and database 
import streamlit as st
from backend_mar import AadhaarMarketingDB

# --- Database Connection ---
# Replace these with your actual PostgreSQL credentials
DB_NAME = "aadhaar_marketing"
DB_USER = "your_username"
DB_PASSWORD = "your_password"
DB_HOST = "localhost"
DB_PORT = "5432"

db = AadhaarMarketingDB(DB_NAME, DB_USER, DB_PASSWORD, DB_HOST, DB_PORT)

st.set_page_config(layout="wide")
st.title("Aadhaar Marketing Application")

# --- Tabs for different sections ---
tab1, tab2, tab3 = st.tabs(["Aadhaar Holders", "Campaigns", "Insights"])

with tab1:
    st.header("Aadhaar Holder Management (CRUD)")
    operation = st.selectbox("Choose a CRUD operation:", ["Create", "Read", "Update", "Delete"])

    if operation == "Create":
        with st.form("create_form"):
            aadhaar_id = st.text_input("Aadhaar ID")
            name = st.text_input("Name")
            age = st.number_input("Age", min_value=0)
            gender = st.selectbox("Gender", ["Male", "Female", "Other"])
            contact = st.text_input("Contact")
            submitted = st.form_submit_button("Create Holder")
            if submitted:
                success, message = db.create_aadhaar_holder(aadhaar_id, name, age, gender, contact)
                if success:
                    st.success(message)
                else:
                    st.error(message)

    elif operation == "Read":
        aadhaar_id = st.text_input("Enter Aadhaar ID to read:")
        if aadhaar_id:
            holder = db.read_aadhaar_holder(aadhaar_id)
            if holder:
                st.write(f"**Aadhaar ID:** {holder[0]}")
                st.write(f"**Name:** {holder[1]}")
                st.write(f"**Age:** {holder[2]}")
                st.write(f"**Gender:** {holder[3]}")
                st.write(f"**Contact:** {holder[4]}")
            else:
                st.warning("Aadhaar holder not found.")

    elif operation == "Update":
        with st.form("update_form"):
            aadhaar_id = st.text_input("Aadhaar ID to update:")
            new_contact = st.text_input("New Contact:")
            submitted = st.form_submit_button("Update Holder")
            if submitted:
                success, message = db.update_aadhaar_holder(aadhaar_id, new_contact)
                if success:
                    st.success(message)
                else:
                    st.error(message)

    elif operation == "Delete":
        with st.form("delete_form"):
            aadhaar_id = st.text_input("Aadhaar ID to delete:")
            submitted = st.form_submit_button("Delete Holder")
            if submitted:
                success, message = db.delete_aadhaar_holder(aadhaar_id)
                if success:
                    st.success(message)
                else:
                    st.error(message)
                
with tab2:
    st.header("Campaign Management (CRUD) - Placeholder")
    st.write("Campaign management features can be implemented here, similar to the Aadhaar Holder section.")

with tab3:
    st.header("Business Insights")
    st.subheader("General Statistics")
    col1, col2 = st.columns(2)
    with col1:
        total_holders = db.get_total_aadhaar_holders()
        st.metric("Total Aadhaar Holders", total_holders)
        
        min_revenue = db.get_min_campaign_revenue()
        st.metric("Min Campaign Revenue", f"₹ {min_revenue:,.2f}" if min_revenue is not None else "N/A")
        
    with col2:
        total_revenue = db.get_total_campaign_revenue()
        st.metric("Total Campaign Revenue", f"₹ {total_revenue:,.2f}" if total_revenue is not None else "N/A")
        
        max_revenue = db.get_max_campaign_revenue()
        st.metric("Max Campaign Revenue", f"₹ {max_revenue:,.2f}" if max_revenue is not None else "N/A")

    st.subheader("Campaign Specific Insights")
    campaign_id_for_avg = st.text_input("Enter Campaign ID for average age:")
    if campaign_id_for_avg:
        avg_age = db.get_average_age_of_campaign_target(campaign_id_for_avg)
        if avg_age:

        import psycopg2

class AadhaarMarketingDB:
    def __init__(self, dbname, user, password, host, port):
        self.conn = None
        self.cursor = None
        self.connect(dbname, user, password, host, port)

    def connect(self, dbname, user, password, host, port):
        """Establishes a connection to the PostgreSQL database."""
        try:
            self.conn = psycopg2.connect(
                dbname=dbname,
                user=user,
                password=password,
                host=host,
                port=port
            )
            self.cursor = self.conn.cursor()
        except psycopg2.Error as e:
            print(f"Error connecting to database: {e}")

    def close(self):
        """Closes the database connection."""
        if self.cursor:
            self.cursor.close()
        if self.conn:
            self.conn.close()

    # --- CRUD Operations ---
    
    def create_aadhaar_holder(self, aadhaar_id, name, age, gender, contact):
        """Creates a new Aadhaar holder record."""
        try:
            self.cursor.execute(
                """INSERT INTO aadhaar_holders (aadhaar_id, name, age, gender, contact) VALUES (%s, %s, %s, %s, %s)""",
                (aadhaar_id, name, age, gender, contact)
            )
            self.conn.commit()
            return True, "Aadhaar holder created successfully."
        except psycopg2.Error as e:
            self.conn.rollback()
            return False, f"Error creating Aadhaar holder: {e}"

    def read_aadhaar_holder(self, aadhaar_id):
        """Reads an Aadhaar holder record."""
        self.cursor.execute(
            """SELECT * FROM aadhaar_holders WHERE aadhaar_id = %s""",
            (aadhaar_id,)
        )
        return self.cursor.fetchone()

    def update_aadhaar_holder(self, aadhaar_id, new_contact):
        """Updates an existing Aadhaar holder's contact information."""
        try:
            self.cursor.execute(
                """UPDATE aadhaar_holders SET contact = %s WHERE aadhaar_id = %s""",
                (new_contact, aadhaar_id)
            )
            self.conn.commit()
            return True, "Aadhaar holder updated successfully."
        except psycopg2.Error as e:
            self.conn.rollback()
            return False, f"Error updating Aadhaar holder: {e}"

    def delete_aadhaar_holder(self, aadhaar_id):
        """Deletes an Aadhaar holder record."""
        try:
            self.cursor.execute(
                """DELETE FROM aadhaar_holders WHERE aadhaar_id = %s""",
                (aadhaar_id,)
            )
            self.conn.commit()
            return True, "Aadhaar holder deleted successfully."
        except psycopg2.Error as e:
            self.conn.rollback()
            return False, f"Error deleting Aadhaar holder: {e}"

    # --- Business Insights ---

    def get_total_aadhaar_holders(self):
        """Returns the total number of Aadhaar holders."""
        self.cursor.execute("SELECT COUNT(*) FROM aadhaar_holders")
        return self.cursor.fetchone()[0]

    def get_total_campaign_revenue(self):
        """Calculates the sum of revenue from all campaigns."""
        self.cursor.execute("SELECT SUM(revenue) FROM campaigns")
        return self.cursor.fetchone()[0]

    def get_average_age_of_campaign_target(self, campaign_id):
        """Calculates the average age of Aadhaar holders targeted by a campaign."""
        self.cursor.execute(
            """SELECT AVG(ah.age) FROM aadhaar_holders ah JOIN campaign_targets ct ON ah.aadhaar_id = ct.aadhaar_id WHERE ct.campaign_id = %s""",
            (campaign_id,)
        )
        return self.cursor.fetchone()[0]

    def get_min_campaign_revenue(self):
        """Finds the minimum revenue from a campaign."""
        self.cursor.execute("SELECT MIN(revenue) FROM campaigns")
        return self.cursor.fetchone()[0]

    def get_max_campaign_revenue(self):
        """Finds the maximum revenue from a campaign."""
        self.cursor.execute("SELECT MAX(revenue) FROM campaigns")
        return self.cursor.fetchone()[0]
            st.write(f"Average age of participants for Campaign '{campaign_id_for_avg}': {avg_age:.2f} years")
        else:
            st.warning("Campaign not found or no participants.")

CREATE DATABASE aadhaar_marketing;

\c aadhaar_marketing;

CREATE TABLE aadhaar_holders (
    aadhaar_id VARCHAR(12) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    age INT,
    gender VARCHAR(50),
    contact VARCHAR(255)
);

CREATE TABLE campaigns (
    campaign_id VARCHAR(50) PRIMARY KEY,
    campaign_name VARCHAR(255) NOT NULL,
    start_date DATE,
    end_date DATE,
    target_demographics TEXT,
    revenue DECIMAL(10, 2)
);

CREATE TABLE campaign_targets (
    campaign_id VARCHAR(50) REFERENCES campaigns(campaign_id),
    aadhaar_id VARCHAR(12) REFERENCES aadhaar_holders(aadhaar_id),
    PRIMARY KEY (campaign_id, aadhaar_id)
);
