Reference Gotcha
================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050629 1154 CEST
   :Tags: php

Through `Planet PHP`_ I saw the
blog entry " `Is PHP staying the language I want to work with?`_ ", for with
comments are cowardly disabled. Although the way classes are handled is
debatable, moaning that PHP 4.4 breaks "return ($ret)" when
returning by reference only shows that the programmer has had no clue
about references in the first place. If you place () around a variable,
you're making it an expression. You can only return variables by
references, not expressions. The return-by-reference in this function
never could have worked as it should have in the first place. Clue:
Don't use "return (<something>)", but just "return
<something>".


.. _`Planet PHP`: http://planet-php.org
.. _`Is PHP staying the language I want to work with?`: http://blog.iworks.at/?/archives/21_Is_PHP_staying_the_language_I_want_to_work_with.html

