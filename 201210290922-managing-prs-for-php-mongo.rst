Managing Pull Requests for the MongoDB PHP driver
=================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-10-29 09:22 Europe/London
   :Tags: blog, php, mongodb
   :Short: managepr

The MongoDB_ PHP Driver is hosted as an Open Source project through GitHub_.
Everybody can check-out the repository and provide patches to this. In order
to make things easier to manage, even Hannes, Jeremy and me, the maintainers
of the extension will only merge code into it through patches.

To streamline the managing of those patches we are using GitHub's Pull Request
(PR) system. Any time we fix a bug, or add a feature, we create a branch of 
our personal fork_ and when we are done, we submit a pull request to the
master repository at http://github.com/mongodb/mongo-php-driver. Before
this pull request gets merged, a few things happen. First of all, we make
sure that there is a related issue in JIRA. If there is no issue, then one
has to be created. Also we verify the patch against our 
`contributing guidelines`_ - including whether the patch can actually
automatically be applied to the branch.

Once the pull request is deemed okay, we review the code and the test cases
that are required to be submitted as part of the pull request. In our case,
I usually review Hannes' patches, and Hannes reviews mine. GitHub's PR system
allows us to easily add comments to bits of the patch which is mighty useful.
Of course, as part of the review process we need to be able to compile the
code and run the test cases as well. For this purpose I have written a
small bash function to help me out::

	function mongoprfetch()
	{
	  git checkout master
	  git fetch origin pull/$1/head:pr/$1
	  git checkout pr/$1
	  git rebase master
	}

This function is run on the command line (in the checkout of
mongodb/mongo-php-driver) with as arguments its Pull Request number::

	mongoprfetch 206

The script will checkout master, fetch the PR as it was a branch from GitHub
and rebases it against master. This allows us to test whether the PR cleanly
applies to the latest code in master. Without the rebasing, the commit history
would also look really complex as this figure shows:

.. image:: /images/content/git-branch-mess.png

When all review comments are resolved, the code compiles and the test cases
succeed, it is time to merge the patch. For this, I am using another small
bash function to help me out::

	function mongoprmerge()
	{
	  git checkout master
	  git merge --no-ff -m "Merged pull request #$1" pr/$1
	  git branch -D pr/$1
	  git push
	}

Because we already have rebased against the latest master branch when running
``mongoprfetch 206``, a merge would mean that we lose the ability to see
which commits belonged to the PR - as Git would simply use "fast forward":

.. image:: /images/content/git-branch-flat.png

We find this undesirable and hence we specify the ``--no-ff`` option to
``git merge`` as you can see in the ``mongoprmerge`` function. This produces
histories that look like:

.. image:: /images/content/git-branch-rebase-no-ff.png

The only thing that we have not automated as part of merging pull requests is
closing the associated Jira tickets as well. 

Right now we have no specific scripts to handle also the stable branch
(``v1.2``) as we have been working hard to reimplement the connection handling
stuff in the PHP driver. I will write more about the changes in the driver, as
well as managing a merging to/from a stable branch in future blog posts.

.. _MongoDB: http://mongodb.org
.. _GitHub: http://github.com
.. _fork: http://github.com/derickr/mongo-php-driver
.. _JIRA: http://jira.mongodb.org/browse/PHP
.. _`contributing guidelines`: https://github.com/derickr/mongo-php-driver/blob/master/CONTRIBUTING.md
