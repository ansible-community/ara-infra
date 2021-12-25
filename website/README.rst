ARA's website
=============

Hosted at `ara.recordsansible.org <https://ara.recordsansible.org>`_, the
website is built with `Hugo <https://gohugo.io/>`_ and installed with the
``website`` role from this project's repository.

The website uses the Hugo theme called `PaperMod <https://github.com/adityatelange/hugo-PaperMod>`_
and it must be checked out as part of building the site.

Development
-----------

Clone repositories somewhere::

    # This repo
    git clone https://git.openstack.org/openstack/ara-infra /usr/local/src/ara-infra
    # The Hugo theme (PaperMod)
    git clone https://github.com/adityatelange/hugo-PaperMod /usr/local/src/themes/PaperMod

Install Hugo following instructions from their `documentation <https://gohugo.io/getting-started/installing/>`_.

Run Hugo's standalone server to get a live preview on ``localhost:1313``::

    hugo server --source /usr/local/src/ara-infra/website --themesDir /usr/local/src/themes
