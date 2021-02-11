# Gorm Migrator

[Russian](./README-ru.md) | **English**

The migrator is used to create and apply / undo migrations. In work uses [Gorm](https://gorm.io/).

## Usage
### Add in project

1. The migrator is copied as a utility into the project. Typically in the `cmd / migrate` subdirectory.
2. In the file `migrate.go` it is necessary to configure the import paths (replace` project` in two places with the name of the project directory).
3. The imported package `models` is expected to export two functions:

```
// GetDB returns an instance of * gorm.DB
func GetDB() *gorm.DB

// GetDBType returns the type of the DBMS ("mysql" or "postgres") from config
func GetDBType() string
```

### First run

The migrator in its work relies on the project model, which uses a config to connect to the database. Therefore, the launch should be carried out from such a place that the config file is available.

For example:
* The `.env` file is located in the same place as` main.go` (package main). Then, being in this directory, you should run the command `go run cmd / migrate / migrate.go -h`.

In the following examples, the abbreviation `migrate -h` will be used instead of a full command like` go run cmd / migrate / migrate.go -h`.

### Сreate migrations

To create migrations, use the `migrate --new` command, as a parameter - the name of the migration. If spaces are found in the migration name, they will be replaced with the `_` symbol. So, for example, the command:

```
migrate --new "alter user table add age"
```

will create a file with a migration template similar to the following in the migrator subdirectory: `migrations/2020_11_05_150521_alter_user_table_add_age.go` .

In the generated migration file, do the following:

1. Determine the type of destruction for migration:
* `DestructiveNo`
* `DestructiveUp` - If migration destroys data only when applied
* `DestructiveDown` - If migration destroys data only on rollback
* `DestructiveFully`- If migration destroys data in both cases

2. Replace the stub in the `Up` and` Down` methods with the real implementation. All interaction with the database should be done using `gorm`, using the transaction instance passed to the` Up` method in the `tx` variable.

### View the list of migrations

To view the list of migrations, use the command `migrate --list` .

The output of this command is usually divided into two parts: the list of applied migrations (Implemented migrations) and the list of new migrations (New migrations).

Applied migrations:
* green - migration was successfully applied, migration code is available;
* red - the migration was successfully applied, but the migration code was not found, therefore, such a migration cannot be rolled back.
Applied migrations are split into blocks (using blank lines). Each block of migrations was executed by one command.

New migrations:
* blue - migration, the creation date of which is located after the creation date of the last performed migration;
* yellow - migration with broken execution history (there are applied migrations with a later creation date), there is a possibility that errors can occur when applying such a migration (usually not).

### Applying / rollback migrations

To apply migrations, use the `migrate --up` command. All new migrations will be applied. If you want to apply only one migration, then add the `--one` flag - only the first migration from the list of new migrations will be applied.

To rollback migrations, use the `migrate --down` command. All migrations of the last block of applied migrations will be undone. If you want to undo only one (last) migration, add the `--one 'flag - only the last migration from the list of applied migrations will be undone.

To perform a data-destructive migration, add the `--force` flag to the command.

### Deploy

In the case of deploying to an installation with several servers, a problem may arise with the parallel launch of migrators, which will interfere with each other (including because they will have the same list of migrations to be performed). If deploying to AWS using the CodeDeploy service, then you can use the `--deploy` flag, which will prevent the migrator from making more than one attempt to apply new migrations within the same deployment.

#### Single transaction

By default, the migrator performs every migration in its transaction. The negative side of this implementation is that if an error occurs in a running migration block, only the current migration will be rolled back, previously performed migrations in this block will remain applied. To change this behavior, there is the `-s` flag - in this case, the block of new migrations will be executed in a single transaction, if an error occurs during the process, the entire block of migrations will be rolled back.

This opportunity should be used with caution, as performing different operations working with the same objects in the database in one transaction in gorm works * strange * (fails).

## About

<img src="https://github.com/rosberry/Foundation/blob/master/Assets/full_logo.png?raw=true" height="100" />

This project is owned and maintained by [Rosberry](http://rosberry.com). We build mobile apps for users worldwide 🌏.

Check out our [open source projects](https://github.com/rosberry), read [our blog](https://medium.com/@Rosberry) or give us a high-five on 🐦 [@rosberryapps](http://twitter.com/RosberryApps).

## License

This project is available under the MIT license. See the LICENSE file for more info.
