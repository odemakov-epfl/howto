# Quick how-to run edX Devstack, XBlocks, LTI, Open edX API

This is quick guide to install
- [Open edX Devstack](http://edx.readthedocs.io/projects/edx-installing-configuring-and-running/en/latest/installation/devstack/install_devstack.html)
- [XBlock flow-control](https://github.com/eduNEXT/flow-control-xblock)
- [LTI Provider](https://github.com/odemakov-epfl/epfl_lti_provider)
- Set up Open EDX as [LTI Consumer](https://github.com/odemakov-epfl/howto)

## Knowledge Prerequisites

- Basic terminal usage.
- Vagrant commands.
- The basics of how Python web applications are built, installed, and deployed.
- How to manage a Linux system.
- GIT commands.

## Software Prerequisites

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://www.vagrantup.com/downloads.html)

## 1. EdX devstack
Devstack is a Vagrant instance designed for local development. It
allows you to quickly modyfy Open edX code and check it imediatly.

Devstack simplifies certain production settings to make development
more convenient. For example, nginx and gunicorn are disabled in
devstack; devstack uses Django’s runserver instead.

### 1.1 Create project directory
```
mkdir howto && cd howto
```

### 1.2 Download box file [here](https://drive.switch.ch/index.php/s/m6UwIr3rHbvfpTm/download).
SHA1 checksum: a7e3fce6d0155cde28e9f3253103f3f66ba3ea54

### 1.3 Add dowloaded box file to the Virtualbox
```
vagrant box add ginkgo-devstack-2017-07-14 vagrant-images_ginkgo-devstack-2017-07-14.box
```

### 1.4 Ensure the nfsd client is running
```
sudo nfsd status
nfsd service is enabled
nfsd is running (pid 313, 8 threads)
```
If the nfsd service is not running, use the nfsd start command.
```
sudo nfsd start
Starting the nfsd service
```

### 1.5 Set the OPENEDX_RELEASE environment variable
```
export OPENEDX_RELEASE="open-release/ginkgo.2"
```

### 1.6 Download the install script.
```
curl -OL https://raw.github.com/edx/configuration/$OPENEDX_RELEASE/util/install/install_stack.sh
https://raw.github.com/edx/configuration/open-release/ginkgo.2/util/install/install_stack.sh
```

### 1.7 Run the install script to create and start the devstack virtual machine.
```
bash install_stack.sh devstack
```

### 1.8 Reboot VM
```
vagrant ssh
sudo reboot
```

### 1.9 Start services in tmux session
```
vagrant ssh
tmux new
sudo su edxapp
paver devstack lms [--fast](will live in tmux session #0)
```
_CNTRL-B C_
```
sudo su edxapp
paver devstack studio [--fast](will live in tmux session #1)
```

### 1.10 Create superuser in order by have access to the admin site
_CNTRL-B C_
```
sudo su edxapp
/edx/bin/python.edxapp /edx/app/edxapp/edx-platform/manage.py lms --settings=devstack createsuperuser --username admin
```
_CNTRL-B D_

## 2. XBlock flow-control

The XBlock specification is a component architecture designed to make
it easier to create new online educational experiences. XBlock was
developed by edX. Developers can focus on component itself no need to
download, install and setup edx-platform.

You can find list of available xblocks here
https://openedx.atlassian.net/wiki/spaces/COMM/pages/43385346/XBlocks+Directory

### 2.1 Install XBlock flow-control
https://github.com/eduNEXT/flow-control-xblock
  
Inside tmux session #2 run following (_CNTRL-B 2_)

```
cd ~
source ~/venvs/edxapp/bin/activate
git clone https://github.com/eduNEXT/flow-control-xblock.git
cd flow-control-xblock
pip install -r requirements.txt
```
  
Inside tmux session #1(Studio) restart studio service.(_CNTRL-B 1_)
```
paver devstack studio --fast
```
_CNTRL-B D_

### 2.2 Configute edX
- From the main page of a specific course, navigate to Settings -> Advanced Settings from the top menu.
- Check for the **advanced_modules** policy key, and add "flow-control" to the policy value list.
- Click the "Save changes" button.
- Add "Flow Control" item from the advanced list to the unit you want to control.


## 3. Learning Tools Interoperability(LTI)

Learning Tools Interoperability is an initiative managed by the IMS Global Learning Consortium to seamlessly integrate learning applications used by instructors into their courses. It includes a standard protocol for establishing a trusted relationship between the tool provider and the Learning Management System, so that students and teachers can have a seamless, integrated experience of using the tool within the context of their course.

### 3.1 LTI Consumer
This is the service that is consuming the tool. Typically this is a Learning Management System (LMS) or user portal. The LTI Consumer provides user information and context to the LTI Tool Provider. Additionally the LTI Consumer provides authentication vouching for the user to the LTI Tool Provider.

### 3.2 LTI Provider
This is the service providing the service to the LTI Consumer. This can be on-premises software or a service that is hosted outside the LTI Consumer.

### 3.3 Standards
- In June 2010 the final specifications for LTI v1.0 are released by the IMS Global Learning Consortium.
- In August 2012 the final specifications for LTI v1.1 are released by the IMS Global Learning Consortium. The ability to pass grades back to the Tool Consumer from the Tool Provider is added.
- In January 2014 the final specifications for LTI v2.0 are released by the IMS Global Learning Consortium. It supports rich and complex REST based two way communication between LTI Consumer and Provider.

### 3.4 Platforms

- Open EdX
  Supports LTI v1.1.1. It can be used both as provider or comsumer in different ways:
  - add external LTI content that is displayed only, such as textbook content that doesn’t require a student response.
  - add external LTI content that requires a student response. An external provider will grade student responses.
  - use the component as a placeholder for syncing with an external grading system.

- Moodle
  Supports LTI as provider and consumer. It is easy to integrate
  Moodle quizes to edX. But it sends grade results by cron. We can't use it in our case(devstack installation accessable only from local machine).

- Blackboard
  Supports LTI as provider and consumer. 

- Code embed
  Supports LTI provider. Doesn't have any auth system. We can only include it to the course page(iframe-way).
  Launch URL:             https://code-embed-lti.herokuapp.com/lti_tool
  Key & secret are random

- KHAN Academy
  Provides only links to videos
  Launch URL:         https://www.edu-apps.org/tool_redirect?id=khan_academy.
  Consumer Key:     	A2kvagYQmzUUrE9A
  Consumer Secret:	HK9aYcy7nFHHvS9Y

- h5p.com
  Provides only "iframe" version(This year should be releases LTI support)

- MIT demo LTI Provider
  https://github.com/mitodl/mit_lti_flask_sample
  MIT demo LTI Provider based on PyLTI(https://github.com/mitodl/pylti) and Flask(http://flask.pocoo.org/).

### 3.5 Set up LTI Provider
https://github.com/odemakov-epfl/epfl_lti_provider
```
sudo su edxapp
cd
git clone https://github.com/odemakov-epfl/epfl_lti_provider.git
cd epfl_lti_provider
virtualenv ~/venvs/lti
source ~/venvs/lti/bin/activate
pip install -r requirements.txt
export FLASK_APP=mit_lti_flask_sample.py
export FLASK_DEBUG=1
flask run --host=0.0.0.0
```

Forward port 5000 from guest to host(Vagrantfile).

### 3.6 Set up LTI consumer

#### 3.6.1 Set up course advanced Settings

- From the main page of a specific course, navigate to Settings ->
  Advanced Settings from the top menu.
- Check for the **advanced_modules** policy key, and add "lti" to the
  policy value list.
- Add to the **LTI Passports**
  "lti_starx_add_demo:\_\_consumer_key\_\_:\_\_lti_secret\_\_" to the value
  list. These key and secret are hard coded in LTI provider.

#### Set up LTI unit
- Add "LTI" item from the advanced list to the unit you want to be LTI'ed.
- Set **LTI ID** as you provided in settings("lti_starx_add_demo")
- Set **LTI URL** to http://0.0.0.0:5000
- Set **Open in New Page** to False.
- Set to True **Request user's username** value. As provider will use
  user name in the greeting.
- Set **Scored** to True.
- Set **Weight** value to 10. As provider has 5 questions 2 points each.
- Click the "Save changes" button.
- Click the "Publish" button.

### 3.7 Check out LTI in LMS

### 3.8 Modify LTI Provider
- Set grade to 1
- Update templates
- Suggestions?

## 4. Open edX API
Open edX includes a suite of APIs that allow you to build applications
that interact with the edX platform.

Open edX APIs use Representational State Transfer (**ReST**) design
principles and support JavaScript Object Notation (**JSON**)
data-interchange format.

APIs use **OAuth 2.0** for authentication. OAuth 2.0 is an open standard
used by many systems that require secure user authentication. See the
OAuth 2.0 Standard for more information.

The standard procedure to start communication with APIs is to get
access token first by sending POST request with **ClientID** and
**ClientSecret** to /oauth2/v1/access_token url. Response will be in JSON
format and will contain **AccessToken** and **Expiration** time or error if
any. Then client'll be able to send other requests to APIs with HTTP
header **Authorization**


Currently the Open edX Platform includes the following APIs.
- Course Catalog API
- Enrollment API(Documentation is not accessible)
- User API(Documentation is not accessible)
- Data Analytics API

### 4.1 Course Catalog API
Course Catalog API used to get list of all the course catalogs,
information about a specific catalog and list of all courses in a
catalog.

#### 4.1.1 Get a List of All Course Catalogs:

Request: `GET /catalog/v1/catalogs/`

Response:
```
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": 1,
            "name": "All Courses",
            "query": "*:*",
            "courses_count": 18,
            "viewers": [
                "username1", "username2"
            ]
        }
    ]
}
```

#### 4.1.2 Get Information about a Specific Catalog

Request: `GET /catalog/v1/catalogs/{id}/`

Response:
```
{
    "id": 1,
    "name": "All Courses",
    "query": "*:*",
    "courses_count": 18,
    "viewers": [
        "username1", "username2"
            ]
}
```

#### 4.1.3 Get a List of All Courses in a Catalog

Request: `GET /catalog/v1/catalogs/{id}/courses/`

Response:
```
{
   "count":123,
   "next":"https://api.edx.org/catalog/v1/catalogs/1/courses/?limit=20&offset=40",
   "previous":"https://api.edx.org/catalog/v1/catalogs/1/courses/?limit=20&offset=0",
   "results":[
      {
         "key":"example_course_key",
         "title":"Title of the Course",
         "short_description":"Short description of course content",
         "full_description":"Longer, more detailed description of course content.",
         "level_type":"Introductory",
         "subjects":[
            {
               "name":"Name of subject"
            }
         ],
         "prerequisites":[

         ],
         "expected_learning_items":[

         ],
         "image":[
            {
               "src":"https://example.com/directory/course_image.jpg",
               "description":"Example image for the Example Title course",
               "height":"300",
               "width":"400"
            }
         ],
         "video":[
            {
               "src":"http://www.youtube.com/watch?v=abcdefghijk",
               "description":null,
               "image":null
            }
         ],
         "owners":[
            {
               "key":"example_institution_key",
               "name":"Example Institution",
               "description":null,
               "logo_image":[
                  {
                     "src":"https://example.com/directory/institution_logo.jpg",
                     "description":null,
                     "height":"200",
                     "width":"200"
                  }
               ],
               "homepage_url":null
            }
         ],
         "sponsors":[

         ],
         "modified":"YYYY-MM-DDTHH:MM:SS.SSSSSSZ",
         "course_runs":[
            {
               "course":"course_number",
               "key":"example_course_key",
               "title":"Title of the Course",
               "short_description":"Short description of course content",
               "full_description":"Longer, more detailed description of course content",
               "start":"YYYY-MM-DDTHH:MM:SSZ",
               "end":"YYYY-MM-DDTHH:MM:SSZ",
               "enrollment_start":"YYYY-MM-DDTHH:MM:SSZ",
               "enrollment_end":"YYYY-MM-DDTHH:MM:SSZ",
               "announcement":null,
               "image":[
                  {
                     "src":"https://example.com/directory/course_image.jpg",
                     "description":null,
                     "height":"200",
                     "width":"300"
                  }
               ],
               "video":null,
               "seats":[
                  {
                     "type":"credit",
                     "price":"100.00",
                     "currency":"USD",
                     "upgrade_deadline":"YYYY-MM-DDTHH:MM:SSZ",
                     "credit_provider":"example institution",
                     "credit_hours":3
                  }
               ],
               "content_language":null,
               "transcript_languages":[

               ],
               "instructors":[

               ],
               "staff":[
                  {
                     "key":"staff_key",
                     "name":"Staff Member Name",
                     "title":"Staff Member Title",
                     "bio":"Example staff member bio.",
                     "profile_image":{
                        "src":"https://example.com/image/staff_member_name.png",
                        "description":null,
                        "height":"150",
                        "width":"150"
                     }
                  }
               ],
               "pacing_type":"instructor_paced",
               "min_effort":null,
               "max_effort":null,
               "modified":"YYYY-MM-DDTHH:MM:SSZ"
            }
         ],
         "marketing_url":"https://example.org/url_for_marketing_materials"
      }
   ]
}
```

### 4.2 Enrollment API
Enrollment API used to view user and course enrollment information
and to enroll a user in a course.

Documentation is not accessible.

### 4.3 User API
User API used to view and update user account and preferences
information.

Documentation is not accessible.

### 4.4 Data Analytics API
Data Analytics API used to view and analyze student activity in your course.

http://edx.readthedocs.io/projects/edx-data-analytics-api/en/latest/index.html
