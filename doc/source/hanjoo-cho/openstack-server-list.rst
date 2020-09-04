=================================
openstack server list 명령어 분석
=================================

optional argument 추가
----------------------

   ``--all`` : 검색할 수 있는 모든 파라미터

.. figure:: images/h-option.png
   :alt: h-option

   h-option

.. code:: python

   # Class ListServer :
   #       def get_parser():

               parser.add_argument(
               '--all',
               action='store_true',
               default=False,
               help=_('List All fields in output'),
           )

.. code:: python

   # Class ListServer :
   # def take_action() :

   elif parsed_args.all:
               if parsed_args.no_name_lookup:
                   columns = (
                       'ID',
                       'Name',
                       'Status',
                       'Networks',
                       'Image ID',
                       'Flavor ID',
                       'OS-EXT-AZ:availability_zone',
                       'os-extended-volumes:volumes_attached',
                       'user_id',
                       'updated',
                       'OS-SRV-USG:launched_at',
                   )
               else:
                   columns = (
                       'ID',
                       'Name',
                       'Status',
                       'Networks',
                       'Image Name',
                       'Flavor Name',
                       'Flavor ID',
                       'OS-EXT-AZ:availability_zone',
                       'os-extended-volumes:volumes_attached',
                       'user_id',
                       'updated',
                       'OS-SRV-USG:launched_at',
                   )
               column_headers = (
                   'ID',
                   'Name',
                   'Status',
                   'Networks',
                   'Image',
                   'Flavor',
                   'Flavor ID',
                   'Availability Zone',
                   'Volumes',
                   'User ID',
                   'Updated',
                   'OS-SRV-USG:launched_at',
               )
               mixed_case_fields = [
                   'OS-EXT-AZ:availability_zone',
                   'OS-SRV-USG:launched_at',
               ]

**Ouput**

.. figure:: images/openstack-server-list---all.png
   :alt: openstack-server-list—all

   openstack-server-list—all

Issue 1
-------

*-c 옵션으로는 한 개의 column 밖에 볼 수가 없다.*

.. code:: shell

   $ openstack server list -c Name ID

.. figure:: images/openstack-server-list--c-Name,ID.png
   :alt: openstack-server-list–c-Name,ID

   openstack-server-list–c-Name,ID

..

   .. rubric:: Feedback
      :name: feedback

   옵션 ``-c`` 를 연속해서 입력하면 다수의 column을 선택할 수 있다.

   .. code:: shell

      $ openstack server list -c ID -c Name

   .. figure:: images/openstack-server-list--c-ID--c-Name.png
      :alt: openstack-server-list–c-ID–c-Name

      openstack-server-list–c-ID–c-Name

Issue 2
-------

``optional argument``\ 에 ``all`` 을 추가 후 ``-c`` 옵션이 없으면 많은
column의 table이 출력

   .. rubric:: Feedback
      :name: feedback-1

   ``optional argument``\ 에 ``--long`` 과 ``--all`` 이 있으면
   사용자에게 혼란을 줍니다. ``--all`` 옵션은 제거하는 것이 좋을 것
   같습니다.

Trace
-----

Code의 부분만 발췌하였습니다. 전체 코드가 아님을 알려드립니다.

.. code:: shell

   $ openstack server list -c Name

Output 출력
~~~~~~~~~~~

output으로 Table이 출력되는 흐름을 trace 했습니다.

.. code:: python

   # /cliff/app.py
   # full_name : 'openstack server list'
   cmd_parser = cmd.get_parser(full_name)
   parsed_args = cmd_parser.parse_args(sub_argv)
   result = cmd.run(parsed_args)

cmd_parser.parse_args
^^^^^^^^^^^^^^^^^^^^^

.. code:: python

   # openstack server list -c Name
   # columns에서 'Name' 추가됨

   # parsed_args :
   Namespace(all=False, all_projects=False, changes_before=None, changes_since=None, columns=['Name'], deleted=False, fit_width=False, flavor=None, formatter='table', host=None, image=None, instance_name=None, ip=None, ip6=None, limit=None, locked=False, long=False, marker=None, max_width=0, name=None, name_lookup_one_by_one=False, no_name_lookup=False, noindent=False, print_empty=False, project=None, project_domain=None, quote_mode='nonnumeric', reservation_id=None, sort_columns=[], status=None, unlocked=False, user=None, user_domain=None)

cmd.run(parsed_args)
^^^^^^^^^^^^^^^^^^^^

.. code:: shell

   # /osc_lib/command/command.py

   class Command(command.Command):

       def run(self, parsed_args):
           print("args : ",parsed_args.__class__)
           self.log.debug('run(%s)', parsed_args)
           return super(Command, self).run(parsed_args)

.. code:: python

   # super(Command, self).run(parsed_args) 
   #    -> /cliff/display.py

   def run(self, parsed_args):
     parsed_args = self._run_before_hooks(parsed_args)
     self.formatter = self._formatter_plugins[parsed_args.formatter].obj
     column_names, data = self.take_action(parsed_args)
     column_names, data = self._run_after_hooks(parsed_args,
                                                (column_names, data))
     self.produce_output(parsed_args, column_names, data)
     return 0

**column_names, data = self.take_action(parsed_args) 결과**

.. code:: shell

   # openstackclient/compute/v2/server.py 의 Output
   # (('ID', 'Name', 'Status', 'Networks', 'Image', 'Flavor'), <generator object ListServer.take_action.<locals>.<genexpr> at 0x7fc885014ba0>)

   (Pdb) p column_names
   ('ID', 'Name', 'Status', 'Networks', 'Image', 'Flavor')
   (Pdb) p data
   <generator object ListServer.take_action.<locals>.<genexpr> at 0x7fbe731c4d00>

produce_output
''''''''''''''

.. code:: python

   # super(Command, self).run(parsed_args) 
   #    -> /cliff/lister.py

               def produce_output(self, parsed_args, column_names, data):
           if parsed_args.sort_columns and self.need_sort_by_cliff:
               indexes = [column_names.index(c) for c in parsed_args.sort_columns
                          if c in column_names]
               if indexes:
                   data = sorted(data, key=operator.itemgetter(*indexes))
           (columns_to_include, selector) = self._generate_columns_and_selector(
               parsed_args, column_names)
           if selector:
               # Generator expression to only return the parts of a row
               # of data that the user has expressed interest in
               # seeing. We have to convert the compress() output to a
               # list so the table formatter can ask for its length.
               data = (list(self._compress_iterable(row, selector))
                       for row in data)
           self.formatter.emit_list(columns_to_include,
                                    data,
                                    self.app.stdout,
                                    parsed_args,
                                    )
           return 0

::

   (Pdb) p columns_to_include
   ['Name']
   (Pdb) p selector
   [False, True, False, False, False, False]

.. figure:: images/openstack-server-list.png
   :alt: openstack-server-list

   openstack-server-list

selector 변수에는 ``openstack server list`` 명령어 실행 시 출력되는
column들이 순서대로 매칭되어 있습니다.

저는 ``-c Name`` 옵션을 추가했기 때문에 Name 외의 다른 column들은
False로 되어있는 것을 볼 수 있습니다.

formatter.emit_list
                   

.. code:: python

       def emit_list(self, column_names, data, stdout, parsed_args):
           x = prettytable.PrettyTable(
               column_names,
               print_empty=parsed_args.print_empty,
           )
           x.padding_width = 1

           # Add rows if data is provided
           if data:
               self.add_rows(x, column_names, data)

           # Choose a reasonable min_width to better handle many columns on a
           # narrow console. The table will overflow the console width in
           # preference to wrapping columns smaller than 8 characters.
           min_width = 8
           self._assign_max_widths(
               stdout, x, int(parsed_args.max_width), min_width,
               parsed_args.fit_width)

           formatted = x.get_string()
           stdout.write(formatted)
           stdout.write('\n')
           return

**input parameter**

.. code:: shell

   (Pdb) p column_names
   ['Name']
   (Pdb) p data
   <generator object Lister.produce_output.<locals>.<genexpr> at 0x7fbe73159048>
   (Pdb) p stdout
   <_io.TextIOWrapper name='<stdout>' mode='w' encoding='UTF-8'>
   (Pdb) p parsed_args
   Namespace(all=False, all_projects=False, changes_before=None, changes_since=None, columns=['Name'], deleted=False, fit_width=False, flavor=None, formatter='table', host=None, image=None, instance_name=None, ip=None, ip6=None, limit=None, locked=False, long=False, marker=None, max_width=0, name=None, name_lookup_one_by_one=False, no_name_lookup=False, noindent=False, print_empty=False, project=None, project_domain=None, quote_mode='nonnumeric', reservation_id=None, sort_columns=[], status=None, unlocked=False, user=None, user_domain=None)
   (Pdb) n

``prettyTable.PrettyTable`` 명령으로 해당 column_names에 대한 Table이
생성이 되고 ``x.get_string()`` 으로 string 형태의 표가 출력됩니다.

.. code:: shell

   (Pdb) p formatted
   '+-----------+\n| Name      |\n+-----------+\n| joostance |\n| 90000e    |\n| dev       |\n+-----------+'

-C 옵션 Trace
~~~~~~~~~~~~~

``-c`` 옵션이 주어졌을 때 ``parsed_args`` 에 값이 어떻게 들어오는지
trace 해봤습니다.

::

   # /cliff/app.py
   # full_name : 'openstack server list'
   cmd_parser = cmd.get_parser(full_name)
   parsed_args = cmd_parser.parse_args(sub_argv)
   result = cmd.run(parsed_args)

cmd_parser.parse_args(sub_argv)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

   # /usr/lib/python3.6/argparse.py
   # sub_argv (args) : ['-c', 'Name']
           # =====================================
       # Command line argument parsing methods
       # =====================================
       def parse_args(self, args=None, namespace=None):
           args, argv = self.parse_known_args(args, namespace)
           if argv:
               msg = _('unrecognized arguments: %s')
               self.error(msg % ' '.join(argv))
           return args

self.parse_known_args(args, namespace)
''''''''''''''''''''''''''''''''''''''

.. code:: python

       def parse_known_args(self, args=None, namespace=None):
           # args default to the system args
           if args is None:
               args = _sys.argv[1:]

           # default Namespace built from parser defaults
           if namespace is None:
               namespace = Namespace()

           # add any action defaults that aren't present
           for action in self._actions:
               if action.dest is not SUPPRESS:
                   if not hasattr(namespace, action.dest):
                       if action.default is not SUPPRESS:
                           setattr(namespace, action.dest, action.default)

for 문에서 namespace 값을 setting합니다.

.. code:: shell

   (Pdb) p namespace
   Namespace(all=False, all_projects=False, changes_before=None, changes_since=None, columns=[], deleted=False, fit_width=False, flavor=None, formatter='table', host=None, image=None, instance_name=None, ip=None, ip6=None, limit=None, locked=False, long=False, marker=None, max_width=0, name=None, name_lookup_one_by_one=False, no_name_lookup=False, noindent=False, print_empty=False, project=None, project_domain=None, quote_mode='nonnumeric', reservation_id=None, sort_columns=[], status=None, unlocked=False, user=None, user_domain=None)

이어서 parse_known_args…

.. code:: python

           # parse the arguments and exit if there are any errors
           try:
               namespace, args = self._parse_known_args(args, namespace)
               return namespace, args

.. code:: shell

   (Pdb) p namespace
   Namespace(all=False, all_projects=False, changes_before=None, changes_since=None, columns=['Name'], deleted=False, fit_width=False, flavor=None, formatter='table', host=None, image=None, instance_name=None, ip=None, ip6=None, limit=None, locked=False, long=False, marker=None, max_width=0, name=None, name_lookup_one_by_one=False, no_name_lookup=False, noindent=False, print_empty=False, project=None, project_domain=None, quote_mode='nonnumeric', reservation_id=None, sort_columns=[], status=None, unlocked=False, user=None, user_domain=None)

\**_parse_known_args*\* 메소드 호출 후 ``namespace`` 의 ``columns`` 가
변화되었습니다. ``-c Name`` 의 ``Name`` 이 리스트에 추가되었습니다.

해당 ``namespace`` 는 ``parsed_args = cmd_parser.parse_args(sub_argv)``
로 리턴됩니다.

\*\* 좀 더 들어가 볼까요?

\**_parse_known_args*\*

.. code:: python

       def _parse_known_args(self, arg_strings, namespace):
           # replace arg strings that are file references
           if self.fromfile_prefix_chars is not None:
               arg_strings = self._read_args_from_files(arg_strings)

           # find all option indices, and determine the arg_string_pattern
           # which has an 'O' if there is an option at an index,
           # an 'A' if there is an argument, or a '-' if there is a '--'
           option_string_indices = {}
           arg_string_pattern_parts = []
           arg_strings_iter = iter(arg_strings)
           for i, arg_string in enumerate(arg_strings_iter):

               # all args after -- are non-options
               if arg_string == '--':
                   arg_string_pattern_parts.append('-')
                   for arg_string in arg_strings_iter:
                       arg_string_pattern_parts.append('A')

               # otherwise, add the arg to the arg strings
               # and note the index if it was an option
               else:
                   option_tuple = self._parse_optional(arg_string)
                   if option_tuple is None:
                       pattern = 'A'
                   else:
                       option_string_indices[i] = option_tuple
                       pattern = 'O'
                   arg_string_pattern_parts.append(pattern)

           # join the pieces together to form the pattern
           arg_strings_pattern = ''.join(arg_string_pattern_parts)

           # converts arg strings to the appropriate and then takes the action
           seen_actions = set()
           seen_non_default_actions = set()

.. code:: shell

   (Pdb) p arg_strings
   ['-c', 'Name']
   (Pdb) p arg_string_pattern_parts
   ['O', 'A']
   (Pdb) p arg_strings_pattern
   'OA'

``-옵션`` 은 ‘O’ , ``parameter`` 는 ‘A’ 로 바꿔서
``arg_strings_pattern`` 에 저장됩니다.

.. raw:: html

   <details>

.. raw:: html

   <summary>

[접기]/[펼치기]

.. raw:: html

   </summary>

.. raw:: html

   <h5>

IF ``openstack server list -c ID Name``

.. raw:: html

   </h5>

.. raw:: html

   <pre>
   (Pdb) p arg_string
   '-c'
   (Pdb) p option_tuple
   (_AppendAction(option_strings=['-c', '--column'], dest='columns', nargs=None, const=None, default=[], type=None, choices=None, help='specify the column(s) to include, can be repeated to show multiple columns', metavar='COLUMN'), '-c', None)
   (Pdb) n
   > /usr/lib/python3.6/argparse.py(1821)_parse_known_args()
   -> pattern = 'O'
   <br/>
   (Pdb) p arg_string
   'ID'
   (Pdb) p option_tuple
   None
   (Pdb) n
   > /usr/lib/python3.6/argparse.py(1818)_parse_known_args()
   > -> pattern = 'A'
   <br/>
   (Pdb) p arg_string
   'Name'
   (Pdb) p option_tuple
   None
   (Pdb) n
   > /usr/lib/python3.6/argparse.py(1818)_parse_known_args()
   > -> pattern = 'A'
   (Pdb) p arg_strings_pattern
   'OAA'
   </pre>

.. raw:: html

   </div>

.. raw:: html

   </details>

Issue 1 - 구현
--------------

1) arg_strings_pattern에서 OA, OAOA 뿐 아니라 OAA, OAAA … 도 인식하도록 구현


현재에는 ``-c`` 옵션 사용 시 O 뒤에 하나의 A만 인식하여 Namespace의
column에 넣도록 되어 있습니다.

2) Error 해결


3) 결론


.. figure:: images/openstack-server-list-multi-column-by-one-opt.png
   :alt: openstack-server-list-multi-column-by-one-opt

   openstack-server-list-multi-column-by-one-opt

Issue 2 - 구현
--------------

1) ``openstack server show dev`` 명령어를 분석해 더 많은 column을 선택할 수 있도록 구현해야 합니다.

