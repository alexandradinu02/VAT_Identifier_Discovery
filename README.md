# UK VAT Identifier Discovery

## 1. The problem

The goal of this project was to see if VAT numbers for UK companies can be found using public information available online.

The main problem is that HMRC can check a VAT number if I already have it, but it cannot give me a VAT number based on a company name.

Another difficulty is that not every UK company is VAT registered. Because of this, if I do not find a VAT number, I cannot know for sure if the company does not have one or if my search method simply failed.

For this reason, I focused more on finding correct VAT numbers than on finding as many numbers as possible.

## 2. My approach

I separated the problem into two main steps:

1. Find a possible VAT number.
2. Check that the VAT number is valid and belongs to the correct company.

I used Companies House to create a sample of UK companies.

Then I tested three possible ways of finding VAT numbers:

* public government spending data;
* official company websites;
* EORI numbers.

When I found a VAT candidate, I used HMRC to check it before accepting it as a result.

## 3. Sample

I used the Companies House bulk dataset from August 2026.

The dataset contained 5,695,465 company records. Of these, 5,190,464 had the status `Active`.

I randomly selected 100 active companies before starting the VAT search.

I used a fixed random seed so that the same sample can be created again.

For the manual website and EORI tests, I used the first 20 companies from this sample.

The sample represents active Companies House companies. It does not necessarily represent the exact type of suppliers that a manufacturing company would have.

## 4. Sources tested

### DEFRA public spending data

The first source I tested was a public DEFRA spending dataset.

This dataset was interesting because it already contained supplier names and VAT numbers.

The file contained 468 unique suppliers. 333 of them had a 9-digit VAT candidate after cleaning the VAT values.

This gave a VAT candidate coverage of 71.2% inside the DEFRA supplier dataset.

However, when I compared these suppliers with my random sample of 100 Companies House companies, I found:

**0 exact matches.**

This showed me that DEFRA can contain useful VAT information, but its coverage is limited to the suppliers that appear in that particular dataset.

I also noticed that VAT values were stored in different formats. Some contained `GB`, spaces or punctuation, so they had to be cleaned before they could be compared.

Another finding was that the same VAT number could appear with different supplier names. Some were simple name variations, while other cases could be related to VAT groups.

### Official company websites

Next, I tested official company websites.

I used the first 20 companies from my random sample.

For each company, I tried to identify the correct official website and looked for VAT information on pages such as:

* legal pages;
* terms and conditions;
* contact pages;
* website footers;
* PDFs.

This test showed that finding the correct official website is already difficult for some companies.

I also found some interesting cases.

For **Arlington Group Asset Management Limited**, I found a VAT number in an official company document. However, the value contained only 8 digits and HMRC did not accept it as a correctly formatted UK VAT number. I therefore did not accept it as a verified result.

For **Clarity Financial Limited**, I found a VAT number on the correct company website, but the number actually belonged to another company mentioned on the page. I rejected this result.

For **AstraZeneca UK Limited**, I found the VAT number `582323642` on an official company page.

I checked the number using HMRC.

HMRC confirmed:

* the VAT number was valid;
* the registered business name was `ASTRAZENECA UK LIMITED`;
* the registered address matched the Companies House address.

I therefore accepted this as a verified VAT match.

From the 20 companies tested through the website approach, I found:

**1 fully verified VAT number.**

This gives a verified discovery rate of:

**1 / 20 = 5%.**

The result is small, but it shows that official websites can provide high-confidence VAT information when the number is clearly published.

### 4.3 EORI

I also tested whether EORI numbers could help with VAT discovery.

For VAT-registered UK businesses, the VAT number can be included inside the EORI number.

This means that finding an EORI could potentially provide a VAT candidate.

I searched for public EORI information for the same 20 companies used in the website test.

I could not confidently find an EORI for any of them.

This does not mean that these companies do not have an EORI. It only means that I could not discover one through the public web search used in this experiment.

My conclusion is that EORI can be useful when the number is already available from another source, but it did not work well as a standalone discovery method in my sample.

## 5. Proof of concept results

The main results of the experiment were:

* Companies House sample: **100 active companies**
* DEFRA suppliers: **468 unique suppliers**
* DEFRA suppliers with a 9-digit VAT candidate: **333**
* DEFRA exact matches with my 100-company sample: **0**
* Companies manually tested through websites: **20**
* Fully verified VAT numbers: **1**
* EORI numbers discovered from the 20-company sample: **0**
* Website verified discovery rate: **5%**

The only VAT number I accepted as fully verified was the AstraZeneca result.

I did not accept any VAT number when I could not confidently connect it to the correct company.

Because only one result was finally accepted, the measured false-positive rate among accepted results was:

**0 / 1 = 0%.**

However, this number should be interpreted carefully because the number of verified results is very small.

## 6. What I learned

The biggest difficulty is not checking VAT numbers.

HMRC provides a clear way to check a VAT number once it has already been found.

The difficult part is discovering the number for an arbitrary company.

The sources I tested had different problems:

* DEFRA had useful VAT data but limited company coverage.
* Company websites could provide strong results, but many companies did not publish VAT information or did not have an easily identifiable website.
* EORI was difficult to discover through public search.

I also learned that finding a valid VAT number is not enough. I still need to make sure that it belongs to the correct company.

The Clarity Financial example showed why this is important: a VAT number can appear on the correct website but still belong to another company mentioned on the page.

Another important limitation is that `NOT FOUND` does not mean `NOT VAT REGISTERED`.

It only means that my method did not find a VAT number.

## 7. What I would do with more resources

With more time and resources, I would test the same approach on a much larger number of companies.

### Test more companies

My proof of concept was intentionally small.

I would first run the same process on a larger sample to understand whether the results remain similar.

### Improve website discovery

One of the main problems was finding the correct official website.

I would use company name, Companies House number, address and postcode together to make website identification more reliable.

If I was not confident that a website belonged to the correct company, I would leave the company unresolved.

### Use more sources

I would test more government datasets and commercial company datasets.

I would also automate part of the website search and page checking instead of reviewing every company manually.

### Keep HMRC verification

Every VAT number would still be treated as a candidate until it was checked through HMRC.

I would compare the business name and address returned by HMRC with Companies House.

If the match was unclear, I would prefer to leave the VAT missing instead of assigning the wrong number.

### Cost

Based on this small experiment, I cannot give a reliable cost per company.

I would first run a larger pilot and measure how much automated work and manual review is needed for each company.

The most expensive cases would probably be companies where the correct website cannot be identified or where the VAT match is unclear.

### What would probably be difficult at scale

Based on my sample, I think the first major problem would be identifying the correct official website.

Other problems would include:

* websites that do not publish VAT information;
* old or incorrect information;
* VAT groups;
* companies with no public VAT information;
* cases that require manual review.

### What I would monitor

I would mainly monitor:

* how many companies have an identified official website;
* how many produce a VAT candidate;
* how many candidates pass HMRC verification;
* how many are rejected because the company information does not match;
* how many cases need manual review.

## 8. Debate topics

### Could VAT numbers be generated and checked against HMRC?

In theory, possible VAT numbers could be generated using the UK VAT format and checksum and then tested with HMRC.

I would not use this as my main discovery method.

Even if I found a valid VAT number, I would still need to know which company it belongs to. It could also require a very large number of checks.

I think HMRC is more useful for checking VAT candidates found from other sources.

### How would I keep the dataset current?

VAT registrations can change, so the dataset would need to be checked regularly.

I would store the date when each VAT number was last verified and periodically check the numbers again.

I would also pay more attention to companies whose name, address or status changes in Companies House.

### How would I know if the dataset was wrong?

There is no complete reference dataset, so it would be difficult to identify every missing or incorrect VAT number.

I would use HMRC verification and regularly review a sample of accepted matches.

If the company name or address returned by HMRC did not match Companies House, I would investigate the result instead of accepting it automatically.

### Which sources would I not be comfortable using in a commercial product?

I would be careful with third-party company directories when it is not clear where their VAT information comes from or how recent it is.

I would also not automatically accept VAT numbers found on webpages.

During my test, I found a VAT number on the correct company website that actually belonged to another company mentioned on the page.

For a commercial dataset, I would prefer information that can be traced back to a clear source and I would still verify VAT candidates through HMRC.

## 9. Conclusion

My small experiment shows that a UK company-to-VAT dataset can be partially built using public web sources.

However, I did not find one source that could provide good coverage for arbitrary UK companies.

The main challenge is VAT discovery, not VAT verification.

A practical solution would probably need several different discovery sources, followed by strict HMRC verification.

Based on what I found, I would prefer to return no VAT number when the evidence is unclear rather than risk assigning the wrong VAT number to a company.
