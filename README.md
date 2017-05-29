# Fundamental concepts and tutorials for UIDAI
** Unofficial, unauthorized, personal project. Use at your own risk. Any feedback/improvements are welcome **

## What is Aadhaar / UIDAI ?
Aadhaar is a 12 digit unique-identity number issued to all Indian residents based on their biometric and demographic data. The data is collected by the Unique Identification Authority of India (UIDAI), a statutory authority established on 12 July 2016 by the Government of India, under the Ministry of Electronics and Information Technology, under the provisions of the Aadhaar Act 2016

## Purpose of this document
I had plenty of questions when I wanted to develop an application that would use Aadhaar. I could not find reliable, up-to-date information about Aadhaar easily from the UIDAI website via Google (As of May 2017). 

I made this github project to capture my learnings with the hope that it may be useful for others too. This is not an official document and is not endorsed by anyone in the Government of India. It is purely a personal hobby project. 

If you find any mistakes or feedback, feel free to raise an issue or send a Pull Request with improvements or corrections. 

All of this document is in the public domain. 

The authors of this document cannot be held responsible for anything that happens by reading or following this document.

## Involved Parties

#### Enrolment Agencies
They add new members to the aadhar database. To become an enrolment agent, please refer to the [UIDAI guidelines](https://uidai.gov.in/enrolment-update/ecosystem-partners/enrolment-agencies.html).

#### CIDR
CIDR stands for Central Identities Data Repository. It is a Government of India organization that is maintaining all the data corresponding to aadhar. The data are all stored in a variety of databases such as mongodb, mysql, etc. with plenty of caching and indexes. The CIDRs are not accessible to everyone (as expected). To access the CIDR, one needs to have certain privileges covered in the next section.

#### ASA
ASA stands for Authentication Service Agency. These are a list of companies, who have registered with the UIDAI so that they can access the CIDR. As of creating this document (May 2017), there are 27 ASAs in India, including but not limited to companies like: Reliance, Vodafone, Bharti Airtel, 	National Informatics Center (nic), Carvy, CAMS, etc. The complete list is available [here](https://uidai.gov.in/images/list_of_live_asa.pdf).

##### Becoming an ASA
It is not simple to become an ASA. As mentioned earlier, there are a few conditions that a company should satisfy to become an ASA. They are:
* Should be a registered Indian company,  under the Indian Companies Act 1956
* Should have an annual turnover of at least Rs. **100 crores** in the last three financial years
* Should have a dedicated, leased-line internet connection from the company's datacenter to the CIDR servers
* There are plenty of other compliance related guidelines too (such as the ability to create and maintain access logs, deployment processes for production vs test, etc.)
* The complete list of all eligibility criteria for becoming an ASA is available [here](https://authportal.uidai.gov.in/home-articles?urlTitle=asa-eligibility-criteria&pageType=authentication)

It is safe to assume that an overwhelming majority of the people reading this document, would not be able to become an ASA. If you have 100 crores annual turnover, you will not be looking at random github projects to become an ASA. So we will move to the next section.

#### AUA and KUA
**AUA** stands for **A**uthentication **U**ser **A**gency. **KUA** stands for e-**K**YC **U**ser **A**gency, where **KYC** stands for **K**now **Y**our **C**ustomer. If you want to start a company / project which wants to make use of Aadhaar data, it is very likely that you want to get yourself registered as a AUA or KUA. 

We also have the complete list of all registered and live [AUAs](https://uidai.gov.in/images/list_of_live_aua.pdf) and [KUAs](https://uidai.gov.in/images/list_of_live_kua.pdf). If your application wants to just authenticate an user (either by biometrics (such as fingerprint or retina) or by OTP or by demographics (address, etc.)) you can get yourself registered as a AUA. But if you want more details, such as the permanent address, Phone number, etc. of the citizens as in the aadhar database, then you need to register yourself as a KUA.

These AUAs and KUAs can not talk directly with the CIDR servers where the data is stored. These AUAs and KUAs have to talk to one of the ASAs that we covered earlier. 

However, becoming a AUA/KUA is also not straight-forward. First, a company has to become a Sub-AUA(SA) and then become an AUA and then only can become a KUA. To become a Sub-AUA, a company has to make a contract with another AUA. The contract will be overseen by the UIDAI. The Sub-AUA will get a **unique Sub-AUA code**. All the authentication/e-KYC requests of the Sub-AUA will go through the AUA, who then forwards it to an ASA, which in turn gets the data from the CIDR.

##### Sub-AUA to AUA
A company after working as a Sub-AUA and performing at least (10,000 transactions / month for 10 months) or at least (25,000 transactions / month for last 4 months) is eligible to become an AUA. The unique Sub-AUA code can be used to verify the transaction count.

##### AUA to KUA
A company that has worked as an AUA can be promoted to a KUA in one of two tracks, a **Regular track** and a **Fast track**. 

Under the Regular track, a AUA can become a KUA iff: 
* It has a minimum Rupees 2 Crores of paid up capital or annual turnover of minimum rupees 5 Crores during the last financial year
* Performed a minimum of 1 Lakh transactions/month in the last 3 months

Under the Fast track, a AUA can become a KUA iff:
* It has a minimum Rupees 4 Crores of paid up capital or annual turnover of minimum Rupees 10 Crores during the last financial year
* Performed a minimum of 3 Lakh transactions in a maximum period of 3 months

In addition to the above requirements, there are other statutory requirements like, the board of the company / partners should pass a formal resolution expressing their intent to become a AUA/KUA, etc. The complete list of these requirements can be found [here](https://authportal.uidai.gov.in/static/Eligibility_Criteria_for_AUA%20_KUA.pdf).
