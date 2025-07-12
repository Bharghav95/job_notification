import requests
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import datetime

# ===== CONFIG =====
RAPIDAPI_KEY = "58bb774602msh0edceffcdcd2c1bp1ce9ecjsne7c8cc3d0184"
RAPIDAPI_HOST = "linkedin-job-search-api.p.rapidapi.com"

SEARCH_TERMS = ["DevOps Engineer", "AWS DevOps Engineer", "Azure", "Kubernetes", "Platform Engineer", "Cloud"]
COMPANIES = ["Barclays", "HSBC", "Goldman Sachs", "Citi", "JP Morgan", "Wipro", "Infosys", "TSB", "Lloyds", "Capitalone", "Natwest", "Techmahindra", "IBM"]

EMAIL_FROM = "bharghav.govind@gmail.com"
EMAIL_TO = "bharghav.govind@gmail.com"
EMAIL_PASSWORD = "txebetqfgmbcsmro"  # Replace with Gmail App Password
SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587


def search_linkedin_jobs():
    url = f"https://{RAPIDAPI_HOST}/jobs/search"
    headers = {
        "X-RapidAPI-Key": RAPIDAPI_KEY,
        "X-RapidAPI-Host": RAPIDAPI_HOST
    }

    job_results = []

    for role in SEARCH_TERMS:
        for company in COMPANIES:
            querystring = {
                "query": f"{role} {company}",
                "location": "United Kingdom",
                "page": "1"
            }

            response = requests.get(url, headers=headers, params=querystring)

            if response.status_code == 200:
                data = response.json()
                for job in data.get("data", [])[:5]:  # Top 5 jobs per query
                    title = job.get("title", "No title")
                    location = job.get("location", "No location")
                    link = job.get("applyUrl", "No link")
                    company_name = job.get("company", {}).get("name", company)
                    job_results.append(f"{title} at {company_name} ({location})\n{link}\n")
            else:
                job_results.append(f"❌ Error fetching jobs for {role} at {company} (Status {response.status_code})")

    return job_results


def send_email(job_list):
    msg = MIMEMultipart()
    msg['From'] = EMAIL_FROM
    msg['To'] = EMAIL_TO
    msg['Subject'] = f"🛠️  Daily DevOps Jobs - {datetime.datetime.now().strftime('%Y-%m-%d')}"

    body = "\n\n".join(job_list) if job_list else "No new jobs found today."
    msg.attach(MIMEText(body, 'plain'))

    server = smtplib.SMTP(SMTP_SERVER, SMTP_PORT)
    server.starttls()
    server.login(EMAIL_FROM, EMAIL_PASSWORD)
    server.send_message(msg)
    server.quit()


if __name__ == "__main__":
    jobs = search_linkedin_jobs()
    send_email(jobs)
