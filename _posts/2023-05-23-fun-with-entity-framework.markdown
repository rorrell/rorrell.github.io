---
layout: post
title:  "Fun With Entity Framework"
date:   2023-05-23
categories: tutorial
---
![image tooltip here](/assets/maria-teneva-7FmSYQ3Z7fg-unsplash.jpg)

So I've been working with Entity Framework.  This pertains to the [this project](https://github.com/rorrell/CodeHire).  I did a tutorial and followed along and then started my own project.  All was going fine until I...  tried to save child objects to a many to many relationship!  Here are my objects:

```c#
public class JobListing
    {
        public int Id { get; set; }

        public string Title { get; set; }

        ...

        public List<Language> Languages { get; } = new();
    }

    public class Language
    {
        public byte Id { get; set; }

        public string Name { get; set; }

        public List<JobListing> JobListings { get; } = new();
    }
```

This created a relationship table like this:

| LanguageJobListings                 |
|---------------------|---------------|
| Language_Id         | JobListing_Id |

Now perhaps this could have been all fine and good.  The problem was I wanted to use Dtos and refactor into multiple files.  There were the regular controllers and the API controllers with a lot of overlapping code, so I created the business layer, and I decided to break up the business layer into different files for different objects (1 for Job Listings and one for Languages).  The LanguageDto didn't have the list of associated JobListings.  This was all working fine for display purposes, but when I tried creating/editing, I got this error: 

### <font color="red">Cannot insert explicit value for identity column in table 'Languages' when IDENTITY_INSERT is set to OFF.</font>

I searched around on the internet for a long, long time and tried all sorts of things (thank you, git, for letting me undo), but finally I came on an answer in StackOverflow that told me, roughly (sadly I visited so many StackOverflow pages today I can't find the exact quote):

> They have to come from the same context in order to save it to the join table otherwise it will just try to save it to the [Languages] table.

Silly me.  Just be careful refactoring in something like Entity Framework; it might just end up biting you.

That ultimately led me to figure out what the problem was.  It was actually, as best as I can tell, the mapping from the Dto to the original model.  First I map a list of language names to the Dto, using the list of languages from the database.  There's no place on the Dto for me to store the associated list of JobListings for each language.  As a result, when I map them to the original model and add them to the JobListing, then try to save them, Entity Framework is too stupid to recognize them with the JobListings set to null and wants to save them as new Languages but can't do it because they have IDs.  

How did I fix it then?  For now, just to get it working, what I ended up doing was grabbing the Languages from the database a second time in the business logic layer.  That is to make sure the languages are complete so that they can be recognized by the database.  I commented it up just so I don't look at it later, wonder what the heck I was thinking, and try to fix it only to break it.  I tried adding the JobListings to the LanguageDto, and that broke other stuff (presumably because it creates an infinite sort of object with a job listing that has a list of languages that each have a list of job listings that each have a list of languages, etc.), and I just didn't have the patience to figure that out at this time with no guarantee that it would, in fact, be the answer to the problem.  And anyway, what I did seems reasonable for now considering that I only have 5 languages in there and I'm not planning on:

1. Adding the capability of adding languages or
2. Adding any languages directly to the database

Therefore, I have decided to stick with the clunky code I have for the time being.  With so few languages, it runs very fast, and I can, begrudgingly, live with that. 

```c#
            //Controller
            //lbll is the languages business logic layer and calls the database using _context
            //SelectedLanguageNames is a List<string> to which a ListBox saves
            jobListingForm.JobListing.Languages =
                lbll.GetLanguages().Where(
                    l => jobListingForm.SelectedLanguageNames.Contains(l.Name)).ToList()

            //call business logic


            //Business Logic
            var jobListing = Mapper.Map<JobListingDto, JobListing>(jobListingDto);
            jobListing.Languages.Clear();
            var selectedIds = jobListingDto.Languages.
                Select(l => l.Id);
            var langs = _context.Languages.
                Where(l => selectedIds.Contains(l.Id)).ToList();
            jobListing.Languages.AddRange(langs);

            _context.JobListings.Add(jobListing);
            _context.SaveChanges();
```

## Update

I have come up with a better solution.  Instead of going to the database twice, I am passing the SelectedLanguageIds (changed from SelectedLanguageNames) to the business logic layer (as an optional argument so that it does not have to be used this way, in case we are coming from the API instead).  It looks more like this:

```c#
            //Controller - now we don't have the previous controller code at all, 
            //we just pass the SelectedLanguageIds to the business logic layer as an additional parameter
            if (jobListingForm.JobListing.Id == 0)
                bll.CreateJobListing(jobListingForm.JobListing, jobListingForm.SelectedLanguageIds);
            else
            {
                if (!bll.UpdateJobListing(jobListingForm.JobListing.Id, jobListingForm.JobListing, jobListingForm.SelectedLanguageIds))
                    return new HttpNotFoundResult();
            }



            //Business Logic - this is from the edit method, 
            //but the create method is very similar
            if (selectedLanguageIds != null)
            {
                jobListingInDb.Languages.Clear();
                var langs = _context.Languages.
                    Where(l => selectedLanguageIds.Contains(l.Id)).ToList();
                jobListingInDb.Languages.AddRange(langs);
            }
```

I am happy with this solution.  It's more elegant without causing the API to change.  Hopefully you learned something from my journey through this problem.  I certainly did.  Maybe it even helped you with something that's been troubling you.