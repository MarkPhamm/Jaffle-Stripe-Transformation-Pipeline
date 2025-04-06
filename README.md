# Jaffle Stripe Transformation Pipeline

A modern data transformation pipeline that integrates and transforms sample data from **Jaffle Shop** (a fictional e-commerce dataset) and **Stripe** (payment processing data) for analytics. Built with ❤️ using `dbt` (data build tool) and Airflow.

---
![diagram (1)](https://github.com/user-attachments/assets/5318a375-055b-4f1c-8db3-fa99277cd58c)


## 📖 Overview

This pipeline demonstrates how to:
1. **Extract** raw transactional data from Jaffle Shop (sample e-commerce data) and Stripe (payment gateway).
2. **Transform** the data into clean, analysis-ready datasets (e.g., customer lifetime value, payment analytics).
3. **Load** structured data into a data warehouse (e.g., Snowflake).

Ideal for learning data transformation patterns, idempotent modeling, and data quality testing with `dbt`.

## 🔄 Data Flow
![image](https://github.com/user-attachments/assets/9ee49239-d75c-41c3-8d28-88080796f6a7)
### Source Data
1. **Jaffle Shop** (`raw.jaffle_shop`)
   - `customers`: Raw customer information
   - `orders`: Order transactions with status tracking

2. **Stripe** (`raw.stripe`)
   - `payment`: Payment processing data with status and amounts

### Data Models

#### Staging Layer (`models/staging/`)
- **Jaffle Shop**
  - `stg_jaffle_shop__customers`: Cleaned customer data
  - `stg_jaffle_shop__orders`: Standardized order information
- **Stripe**
  - `stg_stripe__payments`: Normalized payment data (amounts converted to dollars)

#### Marts Layer (`models/marts/`)
- **Finance** (`marts/finance/`)
  - `fct_orders`: Order facts with payment amounts
    - Combines order information with successful payments
    - Calculates total order amounts
- **Marketing** (`marts/marketing/`)
  - `dim_customers`: Customer dimension with analytics
    - First and most recent order dates
    - Number of orders
    - Customer lifetime value

## 🧪 Testing & Quality

### Data Tests
1. **Generic Tests**
   - Primary key uniqueness
   - Referential integrity
   - Not null constraints
   - Accepted values validation

2. **Custom Tests**
   - `assert_positive_total_for_payments`: Ensures no negative payment totals

### Data Quality Checks
- **Freshness Checks**
  - Orders: Warn after 12h, error after 24h
  - Payments: Warn after 24h, error after 72h

### Documentation
- Comprehensive field-level documentation
- Value definitions for status fields
- Payment method explanations

## 🛠️ Installation

### Prerequisites
- Python 3.8+
- `dbt-core` and `dbt-snowflake` adapter
- Snowflake account
- Apache Airflow
- Docker (for containerized deployment)

### Steps
1. **Clone the repository**:
   ```bash
   git clone https://github.com/MarkPhamm/Jaffle-Stripe-Transformation-Pipeline.git
   cd Jaffle-Stripe-Transformation-Pipeline
   ```

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up Snowflake connection** in `profiles.yml`:
   ```yaml
   jaffle_shop:
     target: dev
     outputs:
       dev:
         type: snowflake
         account: [your_account]
         user: [username]
         password: [password]
         role: [role]
         database: raw
         warehouse: [warehouse]
         schema: dbt_schema
   ```

4. **Configure Airflow Connection**:
   - Connection ID: `snowflake_conn`
   - Connection Type: `Snowflake`
   - Schema: `dbt_schema`
   - Login: Your Snowflake username
   - Password: Your Snowflake password
   - Extra: Configure account, warehouse, database, and role

5. **Run the project in Docker**:
   - Build the Docker image:
     ```bash
     docker build -t jaffle-stripe-pipeline .
     ```
   - Run the Docker container:
     ```bash
     docker run -d -p 8080:8080 jaffle-stripe-pipeline
     ```
   - Access the Airflow UI at `http://localhost:8080`.

---

## ⚙️ Configuration

1. **Configure Data Sources**:
   - **Jaffle Shop**: Sample data included as CSV files (loaded via `dbt seed`).
   - **Stripe**: Set up Stripe API credentials in `stripe.yml` or environment variables.

2. **Update `dbt_project.yml`**:
   - Adjust database/schema names and materialization settings.

---

## 🚀 Usage

### Local Development
1. **Initialize dbt**:
   ```bash
   cd dbt-dags/dags/dbt/jaffle_stripe
   dbt deps
   dbt seed
   ```

2. **Run transformations**:
   ```bash
   dbt run
   dbt test
   ```

3. **Generate documentation**:
   ```bash
   dbt docs generate
   dbt docs serve
   ```

### Airflow Orchestration
The pipeline is orchestrated using Airflow with the following features:
- Daily schedule (`@daily`)
- Automatic dependency installation
- Snowflake connection management
- Virtual environment isolation

## 📊 Key Metrics

The transformed data enables analysis of:
1. Customer Lifetime Value (CLV)
2. Order Success Rates
3. Payment Method Distribution
4. Order Status Tracking
5. Customer Purchase Patterns

## 🔍 Data Definitions

### Order Status
| Status | Definition |
|--------|------------|
| placed | Order placed, not yet shipped |
| shipped | Order has been shipped |
| completed | Order received by customer |
| return_pending | Return requested |
| returned | Item returned |

### Payment Methods
| Method | Definition |
|--------|------------|
| credit_card | Credit card payment |
| coupon | Discount/promo coupon |
| bank_transfer | Direct bank transfer |
| gift_card | Gift card payment |

## 📚 Resources
- [dbt Documentation](https://docs.getdbt.com/docs/introduction)
- [Airflow Documentation](https://airflow.apache.org/docs/)
- [Snowflake Documentation](https://docs.snowflake.com/)

---

## 📂 Project Structure
```
jaffle-stripe-transformation-pipeline/
├── README.md                   # Project documentation
├── requirements.txt            # Python dependencies
├── .gitignore                 # Git ignore rules
└── dbt-dags/                  # Main project directory
    ├── README.md              # DBT-specific documentation
    ├── Dockerfile             # Container configuration
    ├── requirements.txt       # DBT project dependencies
    ├── airflow_settings.yaml  # Airflow configuration
    ├── dags/                  # Airflow DAGs directory
    │   ├── transformation_dag.py  # Main transformation DAG
    │   └── dbt/              # DBT project files
    │       └── jaffle_stripe/ # Main DBT project
    │           ├── dbt_project.yml    # DBT project configuration
    │           ├── packages.yml       # External package dependencies
    │           ├── models/           # Data transformation models
    │           │   ├── staging/      # Raw data models
    │           │   ├── intermediate/ # Intermediate transformations
    │           │   └── marts/        # Final presentation layer
    │           ├── macros/          # Reusable SQL macros
    │           ├── tests/           # Custom data tests
    │           ├── seeds/           # Static data files
    │           ├── snapshots/       # Type 2 SCD tracking
    │           └── analyses/        # Ad-hoc analyses
    ├── tests/                 # Airflow tests
    ├── plugins/              # Airflow plugins
    └── include/             # Additional resources

```

### 📁 Directory Overview

- **`models/`**: Contains all dbt data models organized in layers:
  - `staging/`: Initial data models that clean and standardize raw data
  - `intermediate/`: Complex transformations and business logic
  - `marts/`: Final presentation layer for business users

- **`macros/`**: Reusable SQL snippets and utility functions
- **`tests/`**: Custom data quality tests and assertions
- **`seeds/`**: Static CSV files for reference data
- **`snapshots/`**: Type 2 Slowly Changing Dimension (SCD) tracking
- **`analyses/`**: One-off analytical queries and explorations

### 🔄 Airflow Integration

The project uses Apache Airflow for orchestration:
- `dags/transformation_dag.py`: Defines the main ETL pipeline
- `airflow_settings.yaml`: Configures Airflow environment
- `Dockerfile`: Sets up containerized deployment

---
