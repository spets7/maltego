
Apologies for the confusion earlier. If you want to compare the contents of the zip files with the files in the other folder, we can modify the code to achieve that. Here's the updated version:

python
Copy code
import os
import zipfile
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

def count_files_in_folder(folder_path):
    files = os.listdir(folder_path)
    count = 0
    for file in files:
        if os.path.isfile(os.path.join(folder_path, file)):
            count += 1
    return count

def get_missing_files(folder1_files, folder2_files):
    folder2_base_names = set(os.path.splitext(file)[0] for file in folder2_files)
    missing_files = []
    for file in folder1_files:
        if os.path.isfile(file):
            base_name = os.path.splitext(file)[0]
            if base_name not in folder2_base_names:
                missing_files.append(file)
    return missing_files

def compare_zip_contents_with_folder(zip_path, folder_path):
    with zipfile.ZipFile(zip_path, 'r') as zip_file:
        zip_file_contents = zip_file.namelist()
    folder_files = os.listdir(folder_path)

    missing_files = []
    for file in zip_file_contents:
        if file not in folder_files:
            missing_files.append(file)
    return missing_files

def send_email(smtp_server, smtp_port, smtp_username, smtp_password, sender_email, recipient_email, subject, message):
    email_message = MIMEMultipart()
    email_message["From"] = sender_email
    email_message["To"] = recipient_email
    email_message["Subject"] = subject
    email_message.attach(MIMEText(message, "plain"))

    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.starttls()
        server.login(smtp_username, smtp_password)
        server.send_message(email_message)

def compare_folders(folder1_path, folder2_path, smtp_server, smtp_port, smtp_username, smtp_password, sender_email, recipient_email):
    folder1_files = os.listdir(folder1_path)
    folder2_files = os.listdir(folder2_path)

    num_folder1_files = count_files_in_folder(folder1_path)
    num_folder2_files = count_files_in_folder(folder2_path)

    missing_files = get_missing_files(folder1_files, folder2_files)

    report_message = f"Number of files in {folder1_path}: {num_folder1_files}\n"
    report_message += f"Number of files in {folder2_path}: {num_folder2_files}\n\n"

    if missing_files:
        report_message += "Missing files in folder2:\n"
        for file in missing_files:
            report_message += file + "\n"

    zip_folder1_files = [file for file in folder1_files if file.endswith('.zip')]
    for zip_file in zip_folder1_files:
        zip_file_path = os.path.join(folder1_path, zip_file)
        missing_files = compare_zip_contents_with_folder(zip_file_path, folder2_path)
        if missing_files:
            report_message += f"\nMissing files in {zip_file}:\n"
            for file in missing_files:
                report_message += file + "\n"

    send_email(smtp_server, smtp_port, smtp_username, smtp_password, sender_email, recipient_email, "Folder Comparison Report", report_message)
    print("Email sent successfully!")

# Example 
compare_folders(
    "/path/to/folder1",
    "/path/to/folder2",
    "your_smtp_server",
    587,
    "your_email@example.com",
    "your_email_password",
    "your_email@example.com

)

