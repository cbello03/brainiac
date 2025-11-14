drop table if exists user;


create table user(
id bigserial primary key,
first_name varchar(60) not null,
last_name varchar(60) not null,
username varchar(30) not null,
create_time date default date(current_timestamp),
change_time date default date(current_timestamp),
);