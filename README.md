# SQLShade is a template system for SQL

SQLShade is Inspired by the *2-Way SQL* idea and extended it(originated by Seasar project’s S2Dao).


## Why SQLShade?

There are a few reasons why SQLShade uses.

1. More Readability
2. More Modularity
3. More Testability


## Supported Features

1. Substitute Statement `/*:var*/'faketext'`
2. If(Conditional) Directive `/*#if var*/ ... /*#endif*/`
3. For(Loop) Directive `/*#for e in list_of_var*/ ... /*#endfor*/`
4. Embed Directive `/*#embed var*/ ... /*#embed*/`


### 1. Substitute Statement

Scalar(=, LIKE, ...):

    SELECT * FROM t_member
    WHERE TRUE
        AND t_member.age = /*:age*/1000
        AND t_member.nickname = /*:nickname*/'my nickname is holder'
        AND t_member.updated_at = /*:updated_at*/CURRENT_TIMESTAMP
        AND t_member.created_at <= /*:created_at*/now()
    ;

    #> bind(age=25, nickname='kjim', created_at=date('2010-06-03'), updated_at=date('2010-06-03'))
    #> -------------------------------------------------------------------------------------------
    #> SELECT * FROM t_member
    #> WHERE TRUE
    #>    AND t_member.age = ?
    #>    AND t_member.nickname = ?
    #>    AND t_member.updated_at = ?
    #>    AND t_member.created_at <= ?
    #> ;

Array(IN):

    SELECT * FROM t_member
    WHERE TRUE
        AND t_member.member_id IN /*:member_id*/(100, 200)
        AND t_member.nickname LIKE /*:nickname*/'%kjim%'
        AND t_member.sex IN /*:sex*/('male', 'female')
    ;

    #> bind(member_id=[3845, 295, 1, 637, 221, 357], nickname='%keiji%', sex=['male', 'female', 'other'])
    #> --------------------------------------------------------------------------------------------------
    #> SELECT * FROM t_member
    #> WHERE TRUE
    #>     AND t_member.member_id IN (?, ?, ?, ?, ?, ?)
    #>     AND t_member.nickname LIKE ?
    #>     AND t_member.sex IN (?, ?, ?)



### 2. If Directive

`True` values:

    (`true`, `1`, `-1`, 'some string', [1, 2, 3], dict(a=1, b=2, c=3)).

`False` values:

    (`false`, `0`, `''`, `[]`, dict()).

Simple:

    SELECT * FROM t_table /*#if order_enabled*/ORDER BY ASC/*#endif*/;

    #> bind(order_enabled=True)
    #> -----------------------
    #> SELECT * FROM t_table ORDER BY ASC;

    #> bind(order_enabled=False)
    #> -----------------------
    #> SELECT * FROM t_table;

True/False keywords:

    SELECT
        fav
        , /*#if false*/id/*#endif*/
        , /*#if false*/updated_at/*#endif*/
        , /*#if false*/created_at/*#endif*/
    FROM
        t_favorite
    ;


### 3. For Directive

Simple List:

    SELECT * FROM t_member
    WHERE TRUE
        /*#for nickname in nicknames*/
        AND (t_member.nickname = /*:nickname*/'')
        AND (t_member.nickname LIKE /*:nickname_global_cond*/'%')
        /*#endfor*/
    ;

    #> bind(nicknames=['kjim', 'keiji'], nickname_global_cond='openbooth')
    #> -------------------------------------------------------------------
    #> SELECT * FROM t_member
    #> WHERE TRUE
    #>
    #>     AND (t_member.nickname = ?)
    #>     AND (t_member.nickname LIKE ?)
    #>
    #>     AND (t_member.nickname = ?)
    #>     AND (t_member.nickname LIKE ?)
    #>
    #> ;

Name Mapped List:

    SELECT * FROM t_member
    WHERE TRUE
        /*#for item in nickname_items*/
        AND (t_member.firstname = /*:item.firstname*/'keiji')
        AND (t_member.lastname = /*:item.lastname*/'muraishi')
        /*#endfor*/
    ;

    #> bind(nickname_items=[
    #>       dict(firstname='keiji', lastname='muraishi'),
    #>       dict(firstname='mba', lastname='apple')
    #> ])
    #> ---------------------------------------------------
    #> SELECT * FROM t_member
    #> WHERE TRUE
    #>
    #>     AND (t_member.firstname = ?)
    #>     AND (t_member.lastname = ?)
    #>
    #>     AND (t_member.firstname = ?)
    #>     AND (t_member.lastname = ?)
    #>
    #> ;

Nested List:

    # TODO


### 4. Embed Directive

Simple:

    SELECT * FROM /*#embed table*/t_table_MOCK/*#endembed*/;

    #> bind(table='t_table_dev')
    #> ---------------------
    #> SELECT * FROM t_table_dev;

    #> bind(table='t_table_prod')
    #> ---------------------
    #> SELECT * FROM t_table_prod;

Embedded Another Tempalte:

    TPL1> SELECT * FROM t_favorite /*#embed orderby*//*#endembed*/;

    TPL2> ORDER BY name ASC

    TPL3> ORDER BY name DESC


    #> TPL1.bind(orderby=TPL2)
    #> -----------------------
    #> SELECT * FROM t_favorite ORDER BY name ASC

    #> TPL1.bind(orderby=TPL3)
    #> -----------------------
    #> SELECT * FROM t_favorite ORDER BY name DESC
