ARA's website
=============

Hosted at `ara.recordsansible.org <https://ara.recordsansible.org>`_, the
website is built with `Hugo <https://gohugo.io/>`_ and installed with the
``ara-website`` role from this project's repository.

The website uses the Hugo theme called `future-imperfect <https://themes.gohugo.io/future-imperfect/>`_
and it must be checked out as part of building the site.

Development
-----------

Clone repositories::

    # This repo
    git clone https://git.openstack.org/openstack/ara-infra /opt/ara-infra
    # The Hugo theme (future-imperfect)
    git clone https://github.com/jpescador/hugo-future-imperfect /opt/hugo/themes/hugo-future-imperfect

Install Hugo following instructions from their `documentation <https://gohugo.io/getting-started/installing/>`_.

Run Hugo's standalone server::

    hugo server --source /opt/ara-infra/website --themesDir /opt/hugo/themes
