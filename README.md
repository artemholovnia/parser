# parser
Parser for OLX.UA

import csv
from bs4 import BeautifulSoup
import requests


global URL
URL = "https://www.olx.ua/list/q-Iphone/"

def get_html(url):
    global html
    r = requests.get(url) 
    html = r.text
    return html

def parse_pages(url,number,city):
    for page in range(1, number + 1):
        print("Page number %i" % (page))
        gener_page = URL + "?page=" + str(page)
        get_html(gener_page)
        get_page(html, city)
    print("Parse process it's OKEY")

def write_csv(offers_dict):
    with open("parseolx.csv", "a") as f:
        writer = csv.writer(f)
        writer.writerow( (offers_dict["DATE"],
                            offers_dict["TITLE"],
                            offers_dict["PRICE"],
                            offers_dict["CITY"],
                            offers_dict["LINK"]) )

def get_page(html, city):
    offers_list = []
    soup = BeautifulSoup(html, "lxml")
    page = soup.find("table", id = "offers_table").find_all("td", class_ = "offer")
    for offers in page:
        try:
            offers_city = offers.find("p", class_ = "marginbott5").find("small", class_ = "x-normal").text.strip()
        except:
            continue
         
        offers_city = offers_city.split(",")[0]
        
        if offers_city == city:
            
            try:
                offers_price = offers.find("p", class_ = "price").text.strip()
            except:
                continue
         
            try:
                offers_title = offers.find("h3", class_ = "x-large").find("a", class_ = "marginright5").text.strip()
            except:
                continue

            try:
                offers_href = offers.find("h3", class_="x-large").find("a", class_="marginright5").get("href").strip()
            except:
                continue

            try:
                offers_date = offers.find("p", class_ = "x-normal").text.strip()
            except:
                continue
                
            offers_dict = {
                           "DATE" : offers_date,
                           "TITLE" : offers_title,
                           "PRICE" : offers_price,
                           "CITY" : offers_city,
                           "LINK" : offers_href}
            
            write_csv(offers_dict)
                

def take_number():
    global number, city
    number = int(input("Take max page number for parse : "))
    city = input("Take city for search : ")
    
    return number, city

take_number()
parse_pages(URL, number, city)
