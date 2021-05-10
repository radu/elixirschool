%{
  version: "2.4.0",
  title: "Basics",
  excerpt: """
  Το Ecto είναι ένα επίσημο Elixir project το οποίο παρέχει μια επικάλυψη της βάσης δεδομένων και μια ενσωματωμένη γλώσσα ερωτημάτων.
Με το Ecto είμαστε σε θέση να δημιουργούμε μετατροπές, να ορίζουμε σχήματα, να εισάγουμε και επεξεργαζόμαστε εγγραφές και να δημιουργούμε ερωτήματα προς αυτές.
  """
}
---

### Προσαρμογείς

Το Ecto υποστηρίζει διαφορετικές βάσεις δεδομένων μέσα από τη χρήση προσαρμογέων.
Μερικά παραδείγματα προσαρμογέων είναι:

* PostgreSQL
* MySQL
* SQLite

Για αυτό το μάθημα θα ρυθμίσουμε το Ecto να χρησιμοποιήσει τον προσαρμογέα PostgreSQL.

### Ξεκινώντας

Στα πλαίσια αυτού του μαθήματος θα καλύψουμε τρία μέρη του Ecto:

* Αποθετήριο - παρέχει τη διεπαφή στη βάση δεδομένων μας, συμπεριλαμβανομένης της σύνδεσης
* Μετατροπές - ένας μηχανισμός για τη δημιουργία, μετατροπή και διαγραφή πινάκων και δεικτών
* Σχήματα - εξειδικευμένες δομές που αναπαριστούν εγγραφές σε πίνακα της βάσης δεδομένων

Για να ξεκινήσουμε θα δημιουργήσουμε ένα νέο project με δέντρο επιτήρησης.

```shell
$ mix new friends --sup
$ cd friends
```

Προσθέστε τα πακέτα εξαρτήσεων ecto_sql και postgrex στο `mix.exs` αρχείο μας:

```elixir
defp deps do
  [
    {:ecto_sql, "~> 3.2"},
    {:postgrex, "~> 0.15"}
  ]
end
```

Κατεβάστε τις εξαρτήσεις με τη χρήση:

```shell
$ mix deps.get
```

#### Δημιουργία ενός Αποθετηρίου

Ένα αποθετήριο στο Ecto δείχνει σε μια πηγή δεδομένων όπως είναι μια βάση δεδομένων Postgres.
Όλες οι επικοινωνίες με τη βάση δεδομένων θα πραγματοποιούνται με τη χρήση αυτού του αποθετηρίου.

Ορίστε ένα αποθετήριο τρέχοντας την εντολή:

```shell
$ mix ecto.gen.repo -r Friends.Repo
```

Αυτή η εντολή θα δημιουργήσει τις απαραίτητες ρυθμίσεις στο αρχείο `config/config.exs` για τη σύνδεση στη βάση δεδομένων, συμπεριλαμβανομένου του προσαρμογέα που θα χρησιμοποιηθεί.
Αυτές είναι οι ρυθμίσεις για την εφαρμογή μας `Friends`:

```elixir
config :friends, Friends.Repo,
  database: "friends_repo",
  username: "postgres",
  password: "",
  hostname: "localhost"
```

Οι παραπάνω γραμμές ορίζουν πως το Ecto θα συνδεθεί στη βάση δεδομένων.
Μπορεί να χρειαστεί να ρυθμίσετε τη βάση δεδομένων σας με τα αντίστοιχα διακριτικά.
Επίσης, δημιουργεί μια ενότητα `Friends.Repo` στο αρχείο `lib/friends/repo.ex`.

```elixir
defmodule Friends.Repo do
  use Ecto.Repo, 
    otp_app: :friends,
    adapter: Ecto.Adapters.Postgres
end
```

Θα χρησιμοποιήσουμε την ενότητα `Friends.Repo` για να στέλνουμε ερωτήματα στη βάση δεδομένων.
Επίσης λέμε σε αυτή την ενότητα να βρει τις πληροφορίες ρύθμισης στην εφαρμογή Elixir `:friends` και ότι επιλέγουμε τον προσαρμογέα `Ecto.Adapters.Postgres`.

Στη συνέχεια, θα ρυθμίσουμε την `Friends.Repo` σαν επιτηρητή μέσα στο δέντρο επιτήρησης της εφαρμογής μας στο `lib/friends/application.ex`.
Αυτό θα ξεκινήσει τη διεργασία Ecto όταν ξεκινάει η εφαρμογή μας.

```elixir
  def start(_type, _args) do
    # List all child processes to be supervised
    children = [
      Friends.Repo,
    ]
  ...
```

Μετά από αυτό θα χρειαστούμε την παρακάτω γραμμή στο αρχείο `config/config.exs`:

```elixir
config :friends, ecto_repos: [Friends.Repo]
```

Αυτό θα επιτρέψει στην εφαρμογή μας να τρέξει εντολές ecto από τη γραμμή εντολών.

Μόλις τελειώσαμε τη ρύθμιση του αποθετηρίου μας!
Τώρα μπορούμε να δημιουργήσουμε τη βάση δεδομένων μέσα στην Postgres με αυτή την εντολή:

```shell
$ mix ecto.create
```

Το Ecto θα χρησιμοποιήσει τις πληροφορίες στο αρχείο `config/config.exs` για να προσδιορίσει πως να συνδεθεί στην Postgres και τι όνομα να δώσει στη βάση δεδομένων.

Αν δείτε σφάλματα, βεβαιωθείτε ότι οι πληροφορίες ρύθμισης είναι σωστές και ότι η postgres διεργασία στο σύστημά σας τρέχει.

### Μετατροπές

Για να δημιουργήσετε και να αλλάξετε πίνακες στη βάση δεδομένων, το Ecto μας παρέχει τις μετατροπές.
Κάθε μετατροπή περιγράφει ένα σύνολο δράσεων που θα γίνουν στη βάση δεδομένων μας, όπως ποιον πίνακα να δημιουργήσει ή να αλλάξει.

Από τη στιγμή που η βάση δεδομένων μας δεν έχει πίνακες ακόμα, θα χρειαστούμε μια μετατροπή για να προσθέσουμε έναν.
Η συνθήκη στο Ecto είναι να ονοματίζουμε τους πίνακές μας στον πλυθηντικό.
Για την εφαρμογή μας, αν θέλουμε να εκφράσουμε μια οντότητα `person` στη βάση δεδομένων, θα χρειαστούμε ένα πίνακα `people`.

Ο καλύτερος τρόπος να δημιουργήσουμε μετατροπές είναι η εργασία mix `ecto.gen.migration <όνομα μετατροπής>`, έτσι στην περίπτωσή μας θα χρησιμοποιήσουμε:

```shell
$ mix ecto.gen.migration create_people
```

Αυτή η εντολή θα δημιουργήσει ένα νέο αρχείο στο φάκελο `priv/repo/migrations` που θα περιέχει την χρονική σήμανση στο όνομα του.
Αν περιηγηθούμε στο φάκελο και ανοίξουμε τη μετατροπή θα πρέπει να δούμε κάτι σαν αυτό:

```elixir
defmodule Friends.Repo.Migrations.CreatePeople do
  use Ecto.Migration

  def change do

  end
end
```

Ας ξεκινήσουμε αλλάζοντας τη συνάρτηση `change/0` για να δημιουργήσουμε ένα νέο πίνακα `people` με τα πεδία `name` και `age`:

```elixir
defmodule Friends.Repo.Migrations.CreatePeople do
  use Ecto.Migration

  def change do
    create table(:people) do
      add :name, :string, null: false
      add :age, :integer, default: 0
    end
  end
end
```

Όπως μπορείτε να δείτε ορίσαμε επίσης τους τύπους δεδομένων των πεδίων.
Επιπρόσθετα, συμπεριλάβαμε τις επιλογές `null: false` και `default: 0`.

Μεταβείτε στο τερματικό και τρέξτε τη μετατροπή:

```shell
$ mix ecto.migrate
```

### Σχήματα

Τώρα που δημιουργήσαμε τον αρχικό πίνακα μας πρέπει να πούμε στο Ecto περισσότερα για αυτόν, μέρος του οποίου γίνεται μέσα από τα σχήματα.
Ένα σχήμα είναι μια ενότητα που ορίζει αντιστοιχίσεις στα πεδία του υποκείμενου πίνακα στη βάση δεδομένων.

Παρόλο που το Ecto προτιμά τον πλυθηντικό στα ονόματα πινάκων, το σχήμα τυπικά είναι στον ενικό, έτσι θα δημιουργήσουμε ένα σχήμα `Person` το οποίο θα συντροφεύει τον πίνακά μας.

Ας δημιουργήσουμε το σχήμα μας στο `lib/friends/person.ex`:

```elixir
defmodule Friends.Person do
  use Ecto.Schema

  schema "people" do
    field :name, :string
    field :age, :integer, default: 0
  end
end
```

Εδώ μπορούμε να δούμε ότι η ενότητα `Friends.Person` λέει στο Ecto ότι αυτό το σχήμα αναφέρεται στον πίνακα `people` και ότι έχει δύο στήλες: `name`, το οποίο είναι αλφαριθμητικό και `age`, το οποίο είναι ακέραιος με προκαθορισμένη τιμή το `0`.

Ας ρίξουμε μια ματιά στο σχήμα μας ανοίγοντας το στο `iex -S mix` και δημιουργώντας ένα νέο άτομο:

```elixir
iex> %Friends.Person{}
%Friends.Person{age: 0, name: nil}
```

Όπως θα περιμέναμε παίρνουμε μια δομή `Person` με την προκαθορισμένη τιμή εφαρμοσμένη στο πεδίο `age`.
Τώρα ας δημιουργήσουμε ένα "πραγματικό" άτομο:

```elixir
iex> person = %Friends.Person{name: "Tom", age: 11}
%Friends.Person{age: 11, name: "Tom"}
```

Από τη στιγμή που τα σχήματα είναι απλά δομές, μπορούμε να αλληλεπιδράσουμε με τα δεδομένα μας όπως έχουμε συνηθίσει:

```elixir
iex> person.name
"Tom"
iex> Map.get(person, :name)
"Tom"
iex> %{name: name} = person
%Friends.Person{age: 11, name: "Tom"}
iex> name
"Tom"
```

Όμοια, μπορούμε να αναβαθμίσουμε τα σχήματά μας όπως θα κάναμε με οποιοδήποτε άλλο χάρτη ή δομή στην Elixir:

```elixir
iex> %{person | age: 18}
%Friends.Person{age: 18, name: "Tom"}
iex> Map.put(person, :name, "Jerry")
%Friends.Person{age: 18, name: "Jerry"}
```

Στο επόμενο μάθημά μας για τα Σετ Αλλαγών, θα δούμε πως να επιβεβαιώσουμε τις αλλαγές δεδομένων μας και τελικά πως να τις οριστικοποιήσουμε στη βάση δεδομένων μας.