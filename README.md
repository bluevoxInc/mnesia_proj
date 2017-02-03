# MnesiaProj

**Description -- demonstration project to use Amnesia(Elixir wrapper for 
Mnesia in a cluser**

1) Create a sys.config file in project root to specify nodes:

[{kernel,
  [
    {sync_nodes_optional, ['n1@127.0.0.1', 'n2@127.0.0.1']},
    {sync_nodes_timeout, 3000}
  ]}
].

2) Start two nodes and ensure they are connected:
  iex --name n1@127.0.0.1 --erl "-config sys.config" -S mix
  iex --name n2@127.0.0.1 --erl "-config sys.config" -S mix

  iex(n1@127.0.0.1)1> Node.list
  [:"n2@127.0.0.1"]

  iex(n2@127.0.0.1)1> Node.list
  [:"n1@127.0.0.1"]

3) Edit mix.exs to add Amnesia as a new dependency:
  
  defp deps do
      [{:amnesia, github: "meh/amnesia", tag: :master}]
  end

  $ mix deps.get


  
4) Create a new database. See lib/database.ex in this project for the
    definition of Database1.

5) Ensure both nodes are running and talking to each other:
  iex(n1@127.0.0.1)1> Node.list
  [:"n2@127.0.0.1"]
  
  iex(n2@127.0.0.1)1> Node.list
  [:"n1@127.0.0.1"]

6) Stop Amnesia on both nodes:
  iex(n1@127.0.0.1)2> Amnesia.stop
  :stopped
  
  iex(n2@127.0.0.1)2> Amnesia.stop
  :stopped

7) Create a schema (only need to do this from one of the nodes):
  iex(n1@127.0.0.1)4> nodes = [node | Node.list]
  [:"n1@127.0.0.1", :"n2@127.0.0.1"]              
  iex(n1@127.0.0.1)5> Amnesia.Schema.create(nodes)
  :ok                                             

  will create Mnesia.n1@127.0.0.1 and Mnesia.n2@127.0.0.1 in the root folder.

8) Start Amnesia on both nodes:
  iex(n1@127.0.0.1)6> Amnesia.start
  :ok
  
  iex(n2@127.0.0.1)3> Amnesia.start
  :ok

9) Create tables (run from one node only):
  iex(n1@127.0.0.1)6> Database.create!(disk: nodes)
  :ok

  #wait for all nodes to complete
  iex(n1@127.0.0.1)7> :ok = Database.wait(15000)   
  :ok

10) Verify table creation:
  
  iex(n1@127.0.0.1)8>:mnesia.info
  running db nodes   = ['n2@127.0.0.1','n1@127.0.0.1']
  stopped db nodes   = [] 
  master node tables = []
  remote             = []
  ram_copies         = []
  disc_copies        = ['Elixir.Database1','Elixir.Database1.Message',
                        'Elixir.Database1.User',schema]
                        disc_only_copies   = []
                        [{'n1@127.0.0.1',disc_copies},
                                {'n2@127.0.0.1',disc_copies}] = [schema,
                                'Elixir.Database1',
                                'Elixir.Database1.User',
                                'Elixir.Database1.Message']

11) Add some data:

  iex(n2@127.0.0.1)4> use Amnesia
  Amnesia.Helper

  iex(n2@127.0.0.1)5> use Database1
  [Amnesia, Amnesia.Fragment, Exquisite, Database1, Database1.User,
   Database1.User, Database1.Message, Database1.Message]

  iex(n2@127.0.0.1)6>Amnesia.transaction do
  iex(n2@127.0.0.1)6>john = %User{name: "John", email: "john@example.com"} |> User.write
  iex(n2@127.0.0.1)6>richard = %User{name: "Richard", email: "richard@example.com"} |> User.write
  iex(n2@127.0.0.1)6>linus   = %User{name: "Linus", email: "linus@example.com"} |> User.write
  iex(n2@127.0.0.1)6>end

12) Read the data
  --from node 2:
  iex(n2@127.0.0.1)7> Amnesia.transaction do
  ...(n2@127.0.0.1)7> User.read(1)
  ...(n2@127.0.0.1)7> end
  %Database1.User{email: "john@example.com", id: 1, name: "John"}

  --from node 1:
  iex(n1@127.0.0.1)9> Amnesia.transaction do
  ...(n1@127.0.0.1)9> User.read(1)
  ...(n1@127.0.0.1)9> end
  %Database1.User{email: "john@example.com", id: 1, name: "John"}

  iex(n1@127.0.0.1)10>Amnesia.transaction d
  ...(n1@127.0.0.1)10> r = User.where id > 0
  ...(n1@127.0.0.1)10> r |> Amnesia.Selection.values 
  ...(n1@127.0.0.1)10> |> Enum.each &IO.inspect(&1)
  ...(n1@127.0.0.1)10> end

%Database1.User{email: "john@example.com", id: 1, name: "John"}
%Database1.User{email: "richard@example.com", id: 2, name: "Richard"}
%Database1.User{email: "linus@example.com", id: 3, name: "Linus"}
:ok

13) Delete data:

  iex(n2@127.0.0.1)8> Amnesia.transaction do
  ...(n2@127.0.0.1)8> bill = %User{name: "Bill", email: "bill@blahblah.com"} |> User.write   
  ...(n2@127.0.0.1)8> end

  iex(n1@127.0.0.1)11> Amnesia.transaction do
  ...(n1@127.0.0.1)11> bill = %User{name: "Bill", email: "bill@blahblah.com">
  ...(n1@127.0.0.1)11> end

  iex(n1@127.0.0.1)12>Amnesia.transaction d
  ...(n1@127.0.0.1)12> r = User.where id > 0
  ...(n1@127.0.0.1)12> r |> Amnesia.Selection.values
  ...(n1@127.0.0.1)12> |> Enum.each &IO.inspect(&1)
  ...(n1@127.0.0.1)12> end

  %Database1.User{email: "john@example.com", id: 1, name: "John"}
  %Database1.User{email: "richard@example.com", id: 2, name: "Richard"}
  %Database1.User{email: "linus@example.com", id: 3, name: "Linus"}
  %Database1.User{email: "bill@blahblah.com", id: 4, name: "Bill"}
  %Database1.User{email: "bill@blahblah.com", id: 5, name: "Bill"}
  :ok

  iex(n1@127.0.0.1)13> Amnesia.transaction do
  ...(n1@127.0.0.1)13> User.delete(4)
  ...(n1@127.0.0.1)13> end
  :ok

  iex(n1@127.0.0.1)14>Amnesia.transaction d
  142   ...(n1@127.0.0.1)14> r = User.where id > 0
  143   ...(n1@127.0.0.1)14> r |> Amnesia.Selection.values
  144   ...(n1@127.0.0.1)14> |> Enum.each &IO.inspect(&1)
  145   ...(n1@127.0.0.1)14> end

  %Database1.User{email: "john@example.com", id: 1, name: "John"}
  %Database1.User{email: "richard@example.com", id: 2, name: "Richard"}
  %Database1.User{email: "linus@example.com", id: 3, name: "Linus"}
  %Database1.User{email: "bill@blahblah.com", id: 5, name: "Bill"}
  :ok

