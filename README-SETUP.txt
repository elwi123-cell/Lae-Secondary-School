LAE SECONDARY SCHOOL — PRINCIPAL CMS STARTER
============================================

Included:
- admin/login.html                 Principal sign-in page
- admin/dashboard.html             Structured content editor
- admin/admin.css                  Styling for admin pages
- admin/js/supabase-config.js      Supabase public URL and publishable key

PUBLISH TO GITHUB PAGES
1. Extract this ZIP.
2. Copy the "admin" folder into the ROOT of your existing website repository.
   The result should include:
     index.html
     about.html
     ...
     admin/login.html
     admin/dashboard.html
     admin/admin.css
     admin/js/supabase-config.js
3. Commit and push the files to the branch/folder used by GitHub Pages.
4. Open:
   https://elwi123-cell.github.io/Secondary-School-Web-Template-1/admin/login.html

SECURITY / DATABASE EXPECTATIONS
- The Supabase project URL and publishable key are browser-safe values.
- Never add a service_role key, secret key, or database password to these files.
- Public sign-ups should remain disabled.
- Only a trusted administrator should add the Principal's auth user UUID to public.cms_admins.
- Tables expected:
    public.cms_admins(user_id, role)
    public.site_content(id, page_slug, section_key, content, is_published, updated_at, updated_by)
- Row Level Security and table grants must be enabled and tested.

IMPORTANT LIMITATION
This starter editor stores JSON content in Supabase. It does NOT yet rewrite the HTML files in GitHub, and your public pages do not yet automatically display these records. Each public page must be connected to the CMS in a separate integration step while preserving its existing layout.

FIRST TEST
1. Publish the admin folder.
2. Visit /admin/login.html and sign in as the authorized Principal.
3. Create a test record:
   Page filename: index.html
   Section key: cms_connection_test
   Content JSON: {"heading":"CMS connection test","text":"This is a test record."}
   Published: unchecked initially
4. Save, refresh the dashboard, and confirm the record appears.
5. Do not publish sensitive, personal, or confidential information as a test.
6. After confirming the draft saves, we can integrate the homepage to read published content.

TROUBLESHOOTING
- If sign-in redirects or password reset fails, check Supabase Authentication URL Configuration.
- If the login works but the dashboard redirects back, confirm the exact auth user's UUID exists in cms_admins with role='principal'.
- If table access fails, check SQL Editor results, API table grants, and RLS policies. Do not disable RLS to bypass an error.
