---
# Leave the homepage title empty to use the site title
title:
date: 2022-10-24
type: landing

sections:
  - block: slider
    content:
      slides:
      - title: 👋Welcome <br> to <br> **Tech-Learning-Hub**
        content: Take a look👀 at what we're working on...
        align: center
        background:
          image:
            filename: hub.jpg
            filters:
              brightness: 0.8
          position: center
          color: '#5C4033'
      - title: CheatSheets 📚
        content: 'Cheat Sheets To Help You Get Started 🚀'
        align: left
        background:
          image:
            filename: theory.jpeg
            filters:
              brightness: 0.8
          position: center
          color: '#5C4033'
        link:
          icon: compass
          icon_pack: fas
          text: Explore
          url: "#cheatsheets"  
      - title: Latest Projects 📐
        content: 'Let&shy;s build in the :cloud:'
        align: left
        background:
          image:
            # Specify an image from `assets/media/`
            # or delete the image section to remove it
            filename: diy.jpeg
            filters:
              brightness: 0.8
            position: center
            color: '#5C4033'
        link:
          icon: compass
          icon_pack: fas
          text: Explore
          url: "#projects"
      - title: Latest HowTo&shy;s 🤝
        content: 'Follow along and have :smiley:'
        align: left
        background:
          image: 
            filename: tutorial.jpeg
            filters:
              brightness: 0.8
            position: center
            color: '#5C4033'
        link:
          icon: compass
          icon_pack: fas
          text: Explore
          url: "#posts"      
    design:
      # Slide height is automatic unless you force a specific height (e.g. '400px')
      slide_height: ''
      # Make the slides full screen within the browser window?
      is_fullscreen: true
      # Automatically transition through slides?
      loop: false
      # Duration of transition between slides (in ms)
      interval: 2000
      background:
        color: '#5C4033'
        # Name of image in `assets/media/`.
        # image: 
        # filename: open-book.jpg
        # Darken the image? Range 0-1 where 0 is transparent and 1 is opaque.
        # image_darken: 1
        #  Options are `cover` (default), `contain`, or `actual` size.
        # image_size: cover
        # Options include `left`, `center` (default), or `right`.
        # image_position: center
        # Use a fun parallax-like fixed background effect on desktop? true/false
        parallax: true
        # Text color (true=light, false=dark, or remove for the dynamic theme color).
        # text_color_light: false
  
  - block: features
    id: cheatsheets
    content:
      title: 📚 CheatSheets
      items:
        - description:
          icon: fab fa-github
          icon_pack: fab
          name: GitHub
          url: /cheatsheets/github/
        - description:
          icon: fab fa-jenkins
          icon_pack: fab
          name: Jenkins
          url: /cheatsheets/jenkins
        - description:
          icon: "jfrog"
          icon_pack: "custom"
          name: "JFrog"
          url: /post/artifactory
        - description:
          icon: "spring"
          icon_pack: "custom"
          name: "SpringBoot"
          url: /cheatsheets/springboot
        - description:
          icon: fab fa-docker
          icon_pack: fab
          name: Docker
          url: /cheatsheets/docker
        - description:
          icon: "kubernetes"
          icon_pack: "custom"
          name: "Kubernetes"
          url: /cheatsheets/kubernetes
        - description:
          icon: "vagrant"
          icon_pack: "custom"
          name: "Vagrant"
          url: /cheatsheets/vagrant
        - description:
          icon: "splunk"
          icon_pack: "custom"
          name: "Splunk"
          url: /cheatsheets/splunk
        - description:
          icon: "serverless"
          icon_pack: "custom"
          name: "Serverless"
          url: /cheatsheets/serverless
        - description:
          icon: fab fa-aws
          icon_pack: fab
          name: AWS
          url: /cheatsheets/aws
        - description:
          icon: "graphql"
          icon_pack: "custom"
          name: "GraphQL"
          url: /cheatsheets/graphql
        - description: 
          icon: "chatgpt"
          icon_pack: "custom"
          name: "OpenAI"
          url: /cheatsheets/openai
    design:
    # Choose how many columns the section has. Valid values: 1 or 2.
      columns: "1"
      background:
        # color: '#5C4033'
        # Name of image in `assets/media/`.
        image: 
          filename: open-book.jpg
          filters:
          # Darken the image? Range 0-1 where 1 is transparent and 0 is opaque.
            brightness: 1
          # Image fit. Options are `cover` (default), `contain`, or `actual` size.
            size: cover
          # Image focal point. Options include `left`, `center` (default), or `right`.
            position: center
          # Use a fun parallax-like fixed background effect on desktop? true/false
            parallax: true
        # Text color (true=light, false=dark, or remove for the dynamic theme color).
        text_color_light: false
    advanced:
      css_class: fullscreen
  
  - block: portfolio
    id: projects
    content:
      title: 📐Projects
      filters:
        folders:
          - project
      # Default filter index (e.g. 0 corresponds to the first `filter_button` instance below).
      default_button_index: 0
      # Filter toolbar (optional).
      # Add or remove as many filters (`filter_button` instances) as you like.
      # To show all items, set `tag` to "*".
      # To filter by a specific tag, set `tag` to an existing tag name.
      # To remove the toolbar, delete the entire `filter_button` block.
      buttons:
        - name: All
          tag: '*'
        - name: Spring Boot
          tag: Spring Boot
        - name: AWS
          tag: AWS
        - name: Kubernetes
          tag: Kubernetes
        - name: Other
          tag: Demo
    design:
      # Choose how many columns the section has. Valid values: '1' or '2'.
      columns: '2'
      view: showcase
      # For Showcase view, flip alternate rows?
      flip_alt_rows: false
      background:
        # Background color.
        # color: '#5C4033'
        # Background gradient.
        #gradient_start: "DeepSkyBlue"
        #gradient_end: "SkyBlue"
        # Name of image in `assets/media/`.
        image: 
          filename: open-book.jpg
          filters:
          # Darken the image? Range 0-1 where 1 is transparent and 0 is opaque.
            brightness: 1
          # Image fit. Options are `cover` (default), `contain`, or `actual` size.
            size: cover
          # Image focal point. Options include `left`, `center` (default), or `right`.
            position: center
          # Use a fun parallax-like fixed background effect on desktop? true/false
            parallax: true
        # Text color (true=light, false=dark, or remove for the dynamic theme color).
        text_color_light: false

  - block: collection
    id: posts
    content:
      title: 🤝 HowTos
      subtitle: ''
      text: ''
      # Choose how many pages you would like to display (0 = all pages)
      count: 5
      # Filter on criteria
      filters:
        folders:
          - post
        author: ""
        category: ""
        tag: ""
        exclude_featured: false
        exclude_future: false
        exclude_past: false
        publication_type: ""
      # Choose how many pages you would like to offset by
      offset: 0
      # Page order: descending (desc) or ascending (asc) date.
      order: desc
    design:
      # Choose a view for the listings:
      view: Showcase
      columns: '2'
      # For Showcase view, flip alternate rows?
      flip_alt_rows: true
      background:
      # color: '#9fa8da'
      # Name of image in `assets/media/`.
        image: 
          filename: open-book.jpg
          filters:
          # Darken the image? Range 0-1 where 1 is transparent and 0 is opaque.
            brightness: 1
          # Image fit. Options are `cover` (default), `contain`, or `actual` size.
            size: cover
          # Image focal point. Options include `left`, `center` (default), or `right`.
            position: center
          # Use a fun parallax-like fixed background effect on desktop? true/false
            parallax: true
          # Text color (true=light, false=dark, or remove for the dynamic theme color).
            text_color_light: false
    advanced:
      css_class: fullscreen
  
  - block: about.biography
    id: about
    content:
      title: 👦 Hi 👋
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: admin
    design:
      # Choose how many columns the section has. Valid values: '1' or '2'.
      columns: '2'
      background:
        # Background color.
        # color: '#5C4033'
        # Background gradient.
        #gradient_start: "DeepSkyBlue"
        #gradient_end: "SkyBlue"
        # Name of image in `assets/media/`.
        image: 
          filename: biography.jpeg
          filters:
          # Darken the image? Range 0-1 where 1 is transparent and 0 is opaque.
            brightness: 0.6
          # Image fit. Options are `cover` (default), `contain`, or `actual` size.
            size: cover
          # Image focal point. Options include `left`, `center` (default), or `right`.
            position: center
          # Use a fun parallax-like fixed background effect on desktop? true/false
            parallax: true
          # Text color (true=light, false=dark, or remove for the dynamic theme color).
        text_color_light: true
  
  - block: experience
    content:
      title: Experience
      # Date format for experience
      #   Refer to https://wowchemy.com/docs/customization/#date-format
      date_format: Jan 2006
      # Experiences.
      #   Add/remove as many `experience` items below as you like.
      #   Required fields are `title`, `company`, and `date_start`.
      #   Leave `date_end` empty if it's your current employer.
      #   Begin multi-line descriptions with YAML's `|2-` multi-line prefix.
      items:
              
        - title: Principal Software Engineer
          company: Liberty Mutual
          company_url: 'https://www.libertymutual.com/'
          company_logo: liberty-mutual
          location: California
          date_start: '2022-03-28'
          date_end: ''
          description:

        - title: Program Manager
          company: Farmers Insurance
          company_url: 'https://www.farmers.com/'
          company_logo: farmers-insurance
          location: California
          date_start: '2018-11-12'
          date_end: '2022-03-28'
          description:

        - title: Product Consultant - Technical
          company: Cognizant
          company_url: 'https://www.cognizant.com/us/en'
          company_logo: cognizant
          location: New Hampshire
          date_start: '2011-08-29'
          date_end: '2018-11-09'
          description:

        - title: Senior Systems Engineer
          company: Infosys
          company_url: ''
          company_logo: infosys
          location: Ohio
          date_start: '2006-06-19'
          date_end: '2011-08-26'
          description:  
    design:
      columns: '2'
      view: Showcase
      background:
        # Background color.
        # color: '#5C4033'
        # Background gradient.
        #gradient_start: "DeepSkyBlue"
        #gradient_end: "SkyBlue"
        # Name of image in `assets/media/`.
        image: 
          filename: career.jpeg
          filters:
          # Darken the image? Range 0-1 where 1 is transparent and 0 is opaque.
            brightness: 0.6
          # Image fit. Options are `cover` (default), `contain`, or `actual` size.
            size: cover
          # Image focal point. Options include `left`, `center` (default), or `right`.
            position: center
          # Use a fun parallax-like fixed background effect on desktop? true/false
            parallax: true
        # Text color (true=light, false=dark, or remove for the dynamic theme color).
        text_color_light: true
  - block: accomplishments
    content:
      # Note: `&shy;` is used to add a 'soft' hyphen in a long heading.
      title: 'Certifications'
      subtitle:
      # Date format: https://wowchemy.com/docs/customization/#date-format
      date_format: Jan 2006
      # Accomplishments.
      #   Add/remove as many `item` blocks below as you like.
      #   `title`, `organization`, and `date_start` are the required parameters.
      #   Leave other parameters empty if not required.
      #   Begin multi-line descriptions with YAML's `|2-` multi-line prefix.
      items:
        - certificate_url: https://www.credly.com/badges/b9e16b23-78e1-4936-bf0b-23fcda6cdd6d
          date_end: ""
          date_start: "2025-08-10"
          description: ""
          organization: AWS
          organization_url: https://aws.amazon.com/certification/certified-architect-associate/
          title: AWS Certified Architect - Associate
          url: ""
        - certificate_url: https://www.credly.com/badges/eb692716-86d0-40f9-8a35-0acb44d51717/public_url
          date_end: ""
          date_start: "2021-01-25"
          description: ""
          organization: AWS
          organization_url: https://aws.amazon.com/certification/certified-developer-associate/
          title: AWS Certified Developer - Associate
          url: ""
        - certificate_url: https://www.credly.com/badges/2d8b24c0-ece3-4c88-97c2-bfebf2979305/public_url
          date_end: ""
          date_start: "2021-01-06"
          description:
          organization: Scaled Agile
          organization_url: https://www.scaledagile.com/
          title: Certified SAFe® 5 Practitioner
          url: https://www.scaledagile.com/
        - certificate_url:
          date_end:
          date_start: "2018-07-01"
          description: ""
          organization: Guidewire
          organization_url: https://www.guidewire.com/
          title: 'Guidewire Certified Trainer in Insurance Suite 9.x'
          url: ""
    design:
      columns: '2'
      view: Showcase
      background:
        # Background color.
        # color: '#5C4033'
        # Background gradient.
        #gradient_start: "DeepSkyBlue"
        #gradient_end: "SkyBlue"
        # Name of image in `assets/media/`.
        image: 
          filename: certifications.jpeg
          filters:
          # Darken the image? Range 0-1 where 1 is transparent and 0 is opaque.
            brightness: 0
          # Image fit. Options are `cover` (default), `contain`, or `actual` size.
            size: cover
          # Image focal point. Options include `left`, `center` (default), or `right`.
            position: center
          # Use a fun parallax-like fixed background effect on desktop? true/false
            parallax: true
        # Text color (true=light, false=dark, or remove for the dynamic theme color).
        text_color_light: true
  - block: contact
    id: contact
    content:
      title: 📧 Contact
      subtitle: 
      text: |-
        I love learning and teaching new things. If you have questions about my work or see an opportunity to collaborate, please reach out.
      # Contact (add or remove contact options as necessary)
      # email: test@example.org
      # phone: 888 888 88 88
      appointment_url: 'https://calendly.com/cloud-native-wiki/30min'
      # address:
      #  street: 450 Serra Mall
      #  city: Stanford
      #  region: CA
      #  postcode: '94305'
      #  country: United States
      #  country_code: US
      # directions: Enter Building 1 and take the stairs to Office 200 on Floor 2
      # office_hours:
      #   - 'Monday 10:00 to 13:00'
      #   - 'Wednesday 09:00 to 10:00'
      coordinates:
        latitude: '34.1851'
        longitude: '-118.6090'
      contact_links:
        - icon: twitter
          icon_pack: fab
          name: DM Me
          link: 'https://twitter.com/avijit_tweeter'
        - icon: "discord"
          icon_pack: "fab"
          name: "Join the Discord server"
          link: "https://discord.gg/5kf8dzqQFv"
      # Automatically link email and phone or display as text?
      autolink: true
      # Email form provider
      form:
        provider: netlify
        formspree:
          id:
        netlify:
          # Enable CAPTCHA challenge to reduce spam?
          captcha: true
    design:
      columns: '2'
      background:
        # Background color.
        color: '#5C4033'
        # Background gradient.
        #gradient_start: "DeepSkyBlue"
        #gradient_end: "SkyBlue"
        # Name of image in `assets/media/`.
        # Text color (true=light, false=dark, or remove for the dynamic theme color).
        text_color_light: true
        image: 
          filename: contact.jpeg
          filters:
          # Darken the image? Range 0-1 where 1 is transparent and 0 is opaque.
            brightness: 0.8
          # Image fit. Options are `cover` (default), `contain`, or `actual` size.
            size: cover
          # Image focal point. Options include `left`, `center` (default), or `right`.
            position: center
          # Use a fun parallax-like fixed background effect on desktop? true/false
            parallax: true
---
