# Website Information Gathering Tool
+ Code (Python)
    ```python
    import os
    import socket
    from urllib.parse import urlparse
    import requests
    
    # Store login pages globally for later display or export
    login_pages = []
    
    def clear_screen():
        """Clears the terminal/command prompt screen."""
        if os.name == 'nt':  # For Windows
            os.system('cls')
        else:  # For macOS/Linux
            os.system('clear')
    
    def export_to_file(ip_address, domain):
        """Exports the collected information to a text file."""
        with open("info_results.txt", "w") as file:
            file.write("=== Collected Information ===\n")
            
            if domain:
                file.write(f"Domain: {domain}\n")
            else:
                file.write("Domain: Not provided\n")
            
            if ip_address:
                file.write(f"IP Address: {ip_address}\n")
            else:
                file.write("IP Address: Not provided\n")
            
            # Export login pages if found
            if login_pages:
                file.write("\n=== Login Pages ===\n")
                for page in login_pages:
                    file.write(f"{page}\n")
            else:
                file.write("\nNo login pages found.\n")
        
        print("\nInformation has been successfully exported to 'info_results.txt'.")
    
    def get_ip_from_domain(domain):
        """Fetches the IP address of a given domain."""
        try:
            ip = socket.gethostbyname(domain)
            return ip
        except socket.gaierror as e:
            print(f"DNS resolution failed: {e}")  # Debugging output
            return None
    
    def extract_domain_from_url(url):
        """Extracts the domain from a URL."""
        parsed_url = urlparse(url)
        domain = parsed_url.netloc  # e.g., 'g.co', 'example.com'
        
        # In case of a port number (e.g., 'example.com:8080')
        domain = domain.split(':')[0]
        
        # Return just the domain (without any path/query)
        return domain
    
    def find_login_pages(domain):
        """Finds common login pages for a given domain."""
        common_login_paths = [
            '/login', '/signin', '/admin', '/account', '/user/login', '/auth/login', '/login.php', '/admin/login.php'
        ]
        print(f"\nSearching for login pages on {domain}...")
        
        # To prevent checking the same URLs twice (http://example.com/login and https://example.com/login)
        checked_urls = set()
        
        global login_pages
        login_pages = []  # Reset the global list of login pages
        
        # Try HTTPS first, then HTTP if HTTPS fails
        for path in common_login_paths:
            for scheme in ['https', 'http']:  # Try https first, then http if needed
                url = f'{scheme}://{domain}{path}'
                # Check if we already checked this URL (to avoid duplicates)
                if url not in checked_urls:
                    try:
                        print(f"Checking: {url}")  # Debugging output
                        response = requests.get(url, timeout=5, allow_redirects=True)  # Allow redirects
                        if response.status_code == 200:
                            login_pages.append(url)  # Collect valid login pages
                        checked_urls.add(url)  # Mark this URL as checked
                        break  # Stop after the first successful scheme
                    except requests.exceptions.RequestException as e:
                        print(f"Error checking {url}: {e}")  # Debugging output for request errors
    
        # Now print the results (once)
        if not login_pages:
            print("No login pages found.")
        else:
            print(f"\nTotal login pages found: {len(login_pages)}")
            for page in login_pages:
                print(page)
    
    def collect_miscellaneous_info(domain, ip_address):
        """The Miscellaneous menu for sub-domains, login forms, vulnerabilities."""
        while True:
            # Clear screen for Miscellaneous section
            clear_screen()
    
            # Miscellaneous options
            print("=== Miscellaneous ===")
            print("\n1. Sub-Domains")
            print("2. Login Forms")
            print("3. Vulnerabilities")
            print("\n97. Display Collected Information")
            print("98. Export Collected Information")
            print("99. Exit to Main Menu")
            
            # Get user's choice
            choice = input("\nEnter your choice (1-3, 97-99): ")
            
            # Handle user's choice for subdomains, login forms, vulnerabilities
            if choice == "1":
                print("\nFetching Sub-Domains for", domain)
                # Call a function to get subdomains (you can implement it later)
                print("Sub-Domain functionality is not implemented yet.")
                input("\nPress Enter to return to the menu...")
    
            elif choice == "2":
                # Call the function to find login pages
                find_login_pages(domain)
                input("\nPress Enter to return to the menu...")
    
            elif choice == "3":
                print("\nChecking Vulnerabilities for", domain)
                # Call a function to check vulnerabilities (you can implement it later)
                print("Vulnerabilities functionality is not implemented yet.")
                input("\nPress Enter to return to the menu...")
    
            elif choice == "97":
                # Display collected information
                print("\nCollected Information:")
                print(f"Domain: {domain}")
                print(f"IP Address: {ip_address}")
                
                # Display login pages if found
                if login_pages:
                    print("\nLogin Pages Found:")
                    for page in login_pages:
                        print(page)
                else:
                    print("No login pages found.")
                
                input("\nPress Enter to return to the menu...")
                
            elif choice == "98":
                # Export collected information
                export_to_file(ip_address, domain)
                input("\nPress Enter to return to the menu...")
                
            elif choice == "99":
                print("\nReturning to Main Menu.")
                break  # Exit to the main menu
                
            else:
                print("\nInvalid choice. Please choose between 1-3, 97-99.")
                input("\nPress Enter to return to the menu...")
    
    def collect_info():
        """Collects basic information about the target domain."""
        domain = ""
        ip_address = ""
        
        while True:
            # Clear screen for fresh view
            clear_screen()
    
            # Main menu options
            print("=== About Target ===")
            print("\n1. Domain/URL")
            print("\n97. Display Collected Information")
            print("98. Export Collected Information")
            print("99. Exit")
            
            # Prompt user for input
            choice = input("\nEnter your choice (1, 97-99): ")
            
            # Handle the user's choice
            if choice == "1":
                url = input("\nPlease enter the Domain/URL (e.g., https://example.com or example.com/login): ")
    
                # Ensure the URL starts with 'http://' or 'https://' if not provided
                if not url.startswith(('http://', 'https://')):
                    url = 'http://' + url  # Default to 'http://' if no scheme is provided
                
                # Extract domain from the full URL
                domain = extract_domain_from_url(url)
                print(f"Extracted domain: {domain}")  # Debugging output
    
                # Resolve the IP address for the domain
                ip_address = get_ip_from_domain(domain)
                if ip_address:
                    print(f"IP Address for {domain} is {ip_address}")
                else:
                    print(f"Could not resolve the domain: {domain}")
                    
                # Reset login pages before moving to the Miscellaneous section
                global login_pages
                login_pages = []  # Reset the list of login pages
    
                # Now move to the Miscellaneous section
                collect_miscellaneous_info(domain, ip_address)
                
            elif choice == "97":
                # Display collected information
                print("\nCollected Information:")
                print(f"Domain: {domain}")
                print(f"IP Address: {ip_address}")
                
                # Display login pages if found
                if login_pages:
                    print("\nLogin Pages Found:")
                    for page in login_pages:
                        print(page)
                else:
                    print("No login pages found.")
                
                input("\nPress Enter to return to the menu...")
                
            elif choice == "98":
                # Export collected information
                export_to_file(ip_address, domain)
                input("\nPress Enter to return to the menu...")
                
            elif choice == "99":
                print("\nExiting the program. Goodbye!")
                break  # Exit the loop and the program
                
            else:
                print("\nInvalid choice. Please choose between 1-3, 97-99.")
                input("\nPress Enter to return to the menu...")
    
    # Run the program
    collect_info()
    ```
