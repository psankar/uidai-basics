# UIDAI / Aadhar fundamentals
**Unofficial, unauthorized, personal project. Use at your own risk. Any feedback/improvements are welcome**

## What is Aadhaar / UIDAI ?
Aadhaar is a 12 digit unique-identity number issued to all Indian residents based on their biometric and demographic data. The data is collected by the Unique Identification Authority of India (UIDAI), a statutory authority established on 12 July 2016 by the Government of India, under the Ministry of Electronics and Information Technology, under the provisions of the Aadhaar Act 2016

## Purpose of this document
I had plenty of questions when I wanted to develop an application that would use Aadhaar. I could not find reliable, up-to-date information about Aadhaar easily from the UIDAI website via Google (As of May 2017), because there were many broken links and some conflicting texts (across versions). I try to capture all the fundamental knowledge about developing applications on top of UIDAI / Aadhar in a single place.

I made this github project to capture my learnings with the hope that it may be useful for others too. This is not an official document and is **not** endorsed by anyone in the Government of India. It is purely a personal hobby project. 

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

None of the above requirements are applicable for Government of India organizations. For example, state Governments, Banks, PSUs etc. do not have most of the above constraints and can become even ASAs without worrying about 100 crores. There are some differences between Private Companies, Partnership firms, LLPs, Non-profits, etc. Read the official documents in the above links (or even better consult a lawyer) if you want more accurate details.

## Devices, SDKs, APIs etc.
UIDAI relies on [STQC](http://www.stqc.gov.in/) for the certification of devices (such as fingerprint scanners, iris scanners, etc.) that could be used to work with their servers. Most of the device certifications have an expiry date. It means that the manufacturer has to get re-certified. When you are choosing a device, make sure to choose a device that will remain valid for a long time.

The document containing the list of fingerprint scanners that are approved is presently [here](http://www.stqc.gov.in/sites/upload_files/stqc/files/List_BDCS_FPS_Auth_ver3.1.1_23May2017.pdf). This link gets changed often. If you find the link to be invalid, please raise an issue.

Surprisingly (at least to me), even foreign manufactured devices are allowed. In fact, most of the fingerprint scanners seem to be manufactured in Korea, Taiwan, China. There are some devices like one from 3M and one from Mantra MFS100 are made in India.

The devices cannot work without a SDK. Some of the devices have Android SDK. Almost all the devices expect Windows 8 or later for working correctly. Most device SDKs require a .NET runtime. None of the approved scanners work with Mac OS X (afaics). It is strongly recommended that you get a decent windows laptop and a fairly modern Android tablet/phone (beyond Marshmallow) to ensure things work reliably. Most of the manufacturers are greedy and charge extra for the SDK. I have found Mantra to be giving SDKs for free, for even Linux machines (Ubuntu and openSUSE) and they manufacture locally. YMMV.

Once an SDK is available, you can build the aadhaar PID blocks for consuming the uidai APIs. The PID blocks can be built in either XML or with [protobufs](https://developers.google.com/protocol-buffers/) as the data interchange format. The authentication API version 2.0 specification is available [here](https://uidai.gov.in/images/FrontPageUpdates/aadhaar_authentication_api_2_0.pdf). 

All the authentications performed by a Sub-AUA, AUA/KUA, ASA, etc. need to be logged at each level and maintained, for compliance. The fingerprints however should not be stored. It is unclear if/when there will be audits for these logs and what will be the reaction for non-compliance. Any request that is made and is not responded within 24 hours will be cancelled. Any retries for the same request should be made after this window only (Needs citation, heard from a jio agent).

In addition to the fingerprint scanner SDK, there are SDKs provided by some companies, which help you to connect to the UIDAI servers, using their servers/software as the AUA/KUA. So, effectively you become a Sub-AUA/SA to these companies. They charge a fee (typically 1Rupee per authentication, 5Rupees per eKYC, etc.) for each transaction in addition to other subscription charges. This is probably the easiest way for you to make use of aadhar data. However, keep in mind that the aadhar data should never go out of country. If some of these AUAs (for whom you are working as a Sub-AUA) route the traffic to a foreign nation, it is unclear who will be held liable.

## Potential Improvements

UIDAI gives a sample Java client that works against UIDAI that is available [here](https://uidai.gov.in/images/authDoc/uidai-kyc-client-2.0-src.zip). This link is also broken most of the times, I recommend that you visit the [developer site](https://authportal.uidai.gov.in/web/uidai/developer) time to time to see if there are new clients. There are some other sample projects in C/C++ contributed by third parties, but they are all mostly obsolete developed against older version of the API, so I am not linking them here. I really wish that UIDAI provides proper, well-documented REST interfaces for getting these data and also provide official libraries for at least major languages like Java, Python, Go, Javascript, PHP, Ruby, Objective-C/Swift. It is even unclear if we can even contribute these, because the UIDAI sources are not open sourced. 

I could not find an active google group or mailing list either, which will also be a good community initiative. There is a [google group](https://groups.google.com/forum/#!forum/aadhaarauth) existing but it seem to be mostly defunct and new membership request that I gave was not approved even after months.

Also, it is unclear what is the procedure to report discovery of security vulnerabilities if we find any.

## Conclusion
If you like the information and would prefer this to be kept updated, feel free to star the project or give pull-requests.
