-- лр1

-- схемы
CREATE SCHEMA IF NOT EXISTS app;
CREATE SCHEMA IF NOT EXISTS ref;
CREATE SCHEMA IF NOT EXISTS audit;
CREATE SCHEMA IF NOT EXISTS stg;
REVOKE ALL ON SCHEMA public FROM PUBLIC;
REVOKE ALL ON SCHEMA app FROM PUBLIC;
REVOKE ALL ON SCHEMA ref FROM PUBLIC;
REVOKE ALL ON SCHEMA audit FROM PUBLIC;
REVOKE ALL ON SCHEMA stg FROM PUBLIC;
GRANT USAGE ON SCHEMA public TO PUBLIC;

-- таблицы
CREATE TABLE ref.country (
    country_id serial PRIMARY KEY,
    code char(2) NOT NULL UNIQUE CHECK (code ~ '^[A-Z]{2}$'),
    name varchar(100) NOT NULL
);

CREATE TABLE ref.city (
    city_id serial PRIMARY KEY,
    country_id integer NOT NULL REFERENCES ref.country (country_id),
    name varchar(120) NOT NULL,
    UNIQUE (country_id, name)
);

CREATE TABLE ref.booking_status (
    status_id smallserial PRIMARY KEY,
    code varchar(32) NOT NULL UNIQUE,
    name varchar(64) NOT NULL
);

CREATE TABLE ref.payment_status (
    status_id smallserial PRIMARY KEY,
    code varchar(32) NOT NULL UNIQUE,
    name varchar(64) NOT NULL
);

CREATE TABLE app.partner (
    partner_id serial PRIMARY KEY,
    name varchar(200) NOT NULL,
    inn varchar(12) NOT NULL UNIQUE CHECK (inn ~ '^\d{10}(\d{2})?$'),
    contact_email varchar(200) NOT NULL CHECK (contact_email ~* '^[^@]+@[^@]+\.[^@]+$'),
    contact_phone varchar(32) NOT NULL,
    legal_address varchar(300) NOT NULL,
    website varchar(200),
    registration_date date NOT NULL,
    rating numeric(3, 2) NOT NULL DEFAULT 0 CHECK (rating >= 0 AND rating <= 5),
    is_active boolean NOT NULL DEFAULT true,
    created_at timestamptz NOT NULL DEFAULT now()
);
CREATE TABLE app.partner_agreement (
    agreement_id serial PRIMARY KEY,
    partner_id integer NOT NULL REFERENCES app.partner (partner_id),
    agreement_number varchar(64) NOT NULL UNIQUE,
    start_date date NOT NULL,
    end_date date NOT NULL,
    commission_percent numeric(5, 2) NOT NULL CHECK (commission_percent >= 0 AND commission_percent <= 100),
    terms_text text NOT NULL,
    payment_terms_days integer NOT NULL DEFAULT 14 CHECK (payment_terms_days > 0),
    signed_at date NOT NULL,
    is_exclusive boolean NOT NULL DEFAULT false,
    manager_last_name varchar(100) NOT NULL,
	 manager_first_name varchar(100) NOT NULL,
	 manager_middle_name varchar(100),
    CHECK (end_date >= start_date)
);

CREATE TABLE app.route (
    route_id serial PRIMARY KEY,
    name varchar(200) NOT NULL UNIQUE,
    description text NOT NULL,
    duration_days integer NOT NULL CHECK (duration_days > 0),
    start_city_id integer NOT NULL REFERENCES ref.city (city_id),
    end_city_id integer NOT NULL REFERENCES ref.city (city_id),
    transport_type varchar(50) NOT NULL,
    distance_km numeric(10, 2),
    difficulty_level smallint NOT NULL DEFAULT 1 CHECK (difficulty_level BETWEEN 1 AND 5),
    is_active boolean NOT NULL DEFAULT true
);

CREATE TABLE app.tour (
    tour_id serial PRIMARY KEY,
    partner_id integer NOT NULL REFERENCES app.partner (partner_id),
    route_id integer NOT NULL REFERENCES app.route (route_id),
    name varchar(200) NOT NULL,
    description text NOT NULL,
    start_date date NOT NULL,
    end_date date NOT NULL,
    base_price numeric(12, 2) NOT NULL CHECK (base_price > 0),
    currency_code char(3) NOT NULL DEFAULT 'RUB' CHECK (currency_code ~ '^[A-Z]{3}$'),
    max_seats integer NOT NULL CHECK (max_seats > 0),
    meal_plan varchar(50) NOT NULL DEFAULT 'RO',
    is_published boolean NOT NULL DEFAULT true,
    CHECK (end_date >= start_date)
);

CREATE TABLE app.client (
    client_id serial PRIMARY KEY,
    last_name varchar(100) NOT NULL,
    first_name varchar(100) NOT NULL,
    middle_name varchar(100),
    email varchar(200) NOT NULL UNIQUE CHECK (email ~* '^[^@]+@[^@]+\.[^@]+$'),
    phone varchar(32) NOT NULL,
    passport_series varchar(4) NOT NULL CHECK (passport_series ~ '^\d{4}$'),
    passport_number varchar(6) NOT NULL CHECK (passport_number ~ '^\d{6}$'),
    birth_date date NOT NULL CHECK (birth_date < CURRENT_DATE),
    citizenship char(2) NOT NULL DEFAULT 'RU' CHECK (citizenship ~ '^[A-Z]{2}$'),
    created_at timestamptz NOT NULL DEFAULT now(),
    UNIQUE (passport_series, passport_number)
);

CREATE TABLE app.booking (
    booking_id serial PRIMARY KEY,
    client_id integer NOT NULL REFERENCES app.client (client_id),
    status_id smallint NOT NULL REFERENCES ref.booking_status (status_id),
    booked_at timestamptz NOT NULL DEFAULT now(),
    total_amount numeric(12, 2) NOT NULL CHECK (total_amount >= 0),
    discount_percent numeric(5, 2) NOT NULL DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100),
    manager_note text,
    confirmation_code varchar(32) NOT NULL UNIQUE,
    source_channel varchar(50) NOT NULL DEFAULT 'office'
);

CREATE TABLE app.booking_tour (
    booking_id integer NOT NULL REFERENCES app.booking (booking_id),
    tour_id integer NOT NULL REFERENCES app.tour (tour_id),
    seats integer NOT NULL DEFAULT 1 CHECK (seats > 0),
    unit_price numeric(12, 2) NOT NULL CHECK (unit_price >= 0),
    line_amount numeric(12, 2) NOT NULL CHECK (line_amount >= 0),
    discount_percent numeric(5, 2) NOT NULL DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100),
    currency_code char(3) NOT NULL DEFAULT 'RUB' CHECK (currency_code ~ '^[A-Z]{3}$'),
    sequence_no integer NOT NULL DEFAULT 1 CHECK (sequence_no > 0),
    notes text,
    created_at timestamptz NOT NULL DEFAULT now(),
    PRIMARY KEY (booking_id, tour_id),
    UNIQUE (booking_id, sequence_no)
);

CREATE TABLE app.payment (
    payment_id serial PRIMARY KEY,
    booking_id integer NOT NULL REFERENCES app.booking (booking_id),
    status_id smallint NOT NULL REFERENCES ref.payment_status (status_id),
    amount numeric(12, 2) NOT NULL CHECK (amount > 0),
    paid_at timestamptz,
    card_last4 char(4) CHECK (card_last4 IS NULL OR card_last4 ~ '^\d{4}$'),
    card_token varchar(128),
    payer_last_name varchar(100) NOT NULL,
	 payer_first_name varchar(100) NOT NULL,
	 payer_middle_name varchar(100),
    payment_method varchar(50) NOT NULL DEFAULT 'card',
    currency_code char(3) NOT NULL DEFAULT 'RUB' CHECK (currency_code ~ '^[A-Z]{3}$'),
    transaction_ref varchar(64)
);

CREATE TABLE audit.login_log (
    login_id bigserial PRIMARY KEY,
    login_time timestamptz NOT NULL DEFAULT now(),
    username text NOT NULL,
    client_ip inet,
    session_role text NOT NULL,
    application text
);

CREATE TABLE stg.client_import (
    import_id serial PRIMARY KEY,
    last_name varchar(100) NOT NULL,
    first_name varchar(100) NOT NULL,
    middle_name varchar(100),
    email varchar(200) NOT NULL,
    phone varchar(32) NOT NULL,
    passport_series varchar(4) NOT NULL,
    passport_number varchar(6) NOT NULL,
    birth_date date NOT NULL,
    loaded_at timestamptz NOT NULL DEFAULT now(),
    processed boolean NOT NULL DEFAULT false
);
CREATE TABLE stg.session_start (
    start_id bigserial PRIMARY KEY,
    started_at timestamptz NOT NULL DEFAULT now(),
    application text,
    actor text NOT NULL DEFAULT current_user
);

-- индексы
CREATE INDEX idx_city_country ON ref.city (country_id);
CREATE INDEX idx_partner_agreement_partner ON app.partner_agreement (partner_id);
CREATE INDEX idx_tour_partner ON app.tour (partner_id);
CREATE INDEX idx_tour_route ON app.tour (route_id);
CREATE INDEX idx_tour_dates ON app.tour (start_date, end_date);
CREATE INDEX idx_route_cities ON app.route (start_city_id, end_city_id);
CREATE INDEX idx_booking_client ON app.booking (client_id);
CREATE INDEX idx_booking_status ON app.booking (status_id);
CREATE INDEX idx_booking_tour_tour ON app.booking_tour (tour_id);
CREATE INDEX idx_payment_booking ON app.payment (booking_id);
CREATE INDEX idx_login_log_time ON audit.login_log (login_time);
CREATE INDEX idx_login_log_user ON audit.login_log (username);

-- rls
ALTER TABLE app.client ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.client FORCE ROW LEVEL SECURITY;
ALTER TABLE app.payment ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.payment FORCE ROW LEVEL SECURITY;
ALTER TABLE app.partner_agreement ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.partner_agreement FORCE ROW LEVEL SECURITY;
CREATE POLICY client_app_access ON app.client
FOR ALL
TO app_reader, app_writer, app_owner, dml_admin
USING (true)
WITH CHECK (true);

CREATE POLICY payment_app_access ON app.payment
FOR ALL
TO app_reader, app_writer, app_owner, dml_admin
USING (true)
WITH CHECK (true);

CREATE POLICY agreement_app_access ON app.partner_agreement
FOR ALL
TO app_reader, app_writer, app_owner, dml_admin
USING (true)
WITH CHECK (true);

-- тестовые данные
INSERT INTO ref.country (code, name) VALUES
('RU', 'Россия'),
    ('TR', 'Турция'),
    ('EG', 'Египет'),
    ('TH', 'Таиланд'),
    ('IT', 'Италия'),
    ('ES', 'Испания'),
    ('GR', 'Греция'),
    ('AE', 'ОАЭ'),
    ('GE', 'Грузия'),
    ('VN', 'Вьетнам');

INSERT INTO ref.city (country_id, name) VALUES
    (1, 'Москва'),
    (1, 'Санкт-Петербург'),
    (2, 'Анталья'),
    (2, 'Стамбул'),
    (3, 'Хургада'),
    (3, 'Шарм-эш-Шейх'),
    (4, 'Пхукет'),
    (4, 'Бангкок'),
    (5, 'Рим'),
    (6, 'Барселона'),
    (7, 'Афины'),
    (8, 'Дубай');

INSERT INTO ref.booking_status (code, name) VALUES
    ('draft', 'Черновик'),
    ('confirmed', 'Подтверждено'),
    ('paid', 'Оплачено'),
    ('cancelled', 'Отменено'),
    ('completed', 'Завершено');

INSERT INTO ref.payment_status (code, name) VALUES
    ('pending', 'Ожидает'),
    ('authorized', 'Авторизован'),
    ('captured', 'Списан'),
    ('failed', 'Ошибка'),
    ('refunded', 'Возврат');
INSERT INTO app.partner (
    name, inn, contact_email, contact_phone, legal_address, website,
    registration_date, rating, is_active
) VALUES
INSERT INTO app.partner_agreement (
    partner_id, agreement_number, start_date, end_date, commission_percent, terms_text,
    payment_terms_days, signed_at, is_exclusive,
    manager_last_name, manager_first_name, manager_middle_name
) VALUES
INSERT INTO app.route (
    name, description, duration_days, start_city_id, end_city_id,
    transport_type, distance_km, difficulty_level, is_active
) VALUES
INSERT INTO app.tour (
    partner_id, route_id, name, description, start_date, end_date, base_price, currency_code,
    max_seats, meal_plan, is_published
) VALUES
INSERT INTO app.client (
    last_name, first_name, middle_name, email, phone,
    passport_series, passport_number, birth_date, citizenship
) VALUES
    ('Иванов', 'Иван', 'Иванович', 'ivanov@example.com', '+79001110001', '4501', '123456', '1990-01-15', 'RU'),
    ('Петрова', 'Анна', 'Сергеевна', 'petrova@example.com', '+79001110002', '4501', '123457', '1988-03-22', 'RU'),
    ('Сидоров', 'Пётр', 'Алексеевич', 'sidorov@example.com', '+79001110003', '4502', '223456', '1995-07-08', 'RU'),
    ('Козлова', 'Мария', 'Игоревна', 'kozlova@example.com', '+79001110004', '4502', '223457', '1992-11-30', 'RU'),
    ('Новиков', 'Дмитрий', 'Олегович', 'novikov@example.com', '+79001110005', '4503', '323456', '1985-05-12', 'RU'),
    ('Морозова', 'Елена', 'Викторовна', 'morozova@example.com', '+79001110006', '4503', '323457', '1998-09-03', 'RU'),
    ('Волков', 'Андрей', 'Николаевич', 'volkov@example.com', '+79001110007', '4504', '423456', '1991-12-19', 'RU'),
    ('Лебедева', 'Ольга', 'Дмитриевна', 'lebedeva@example.com', '+79001110008', '4504', '423457', '1987-04-25', 'RU'),
    ('Соколов', 'Максим', 'Павлович', 'sokolov@example.com', '+79001110009', '4505', '523456', '1993-08-14', 'RU'),
    ('Зайцева', 'Наталья', 'Романовна', 'zaitseva@example.com', '+79001110010', '4505', '523457', '1996-02-28', 'RU'),
    ('Орлов', 'Кирилл', 'Евгеньевич', 'orlov@example.com', '+79001110011', '4506', '623456', '1989-06-17', 'RU'),
    ('Павлова', 'Дарья', 'Андреевна', 'pavlova@example.com', '+79001110012', '4506', '623457', '1994-10-09', 'RU');

INSERT INTO app.booking (
    client_id, status_id, total_amount, discount_percent,
    manager_note, confirmation_code, source_channel
) VALUES
    (1, 3, 303000.00, 0, NULL, 'CNF-0001', 'office'),
    (2, 2, 125000.00, 5, 'VIP клиент', 'CNF-0002', 'web'),
    (3, 3, 228000.00, 0, NULL, 'CNF-0003', 'office'),
    (4, 2, 196000.00, 0, NULL, 'CNF-0004', 'phone'),
    (5, 1, 145000.00, 0, 'Ожидает оплату', 'CNF-0005', 'web'),
    (6, 3, 336000.00, 10, NULL, 'CNF-0006', 'office'),
    (7, 2, 112000.00, 0, NULL, 'CNF-0007', 'web'),
    (8, 3, 422000.00, 0, NULL, 'CNF-0008', 'office'),
    (9, 3, 135000.00, 0, NULL, 'CNF-0009', 'partner'),
    (10, 2, 180000.00, 0, 'Группа', 'CNF-0010', 'office'),
    (11, 4, 89000.00, 0, 'Отмена по болезни', 'CNF-0011', 'phone'),
    (12, 5, 152000.00, 0, NULL, 'CNF-0012', 'web');

INSERT INTO app.booking_tour (
    booking_id, tour_id, seats, unit_price, line_amount, discount_percent,
    currency_code, sequence_no, notes
) VALUES
    (1, 1, 2, 89000.00, 178000.00, 0, 'RUB', 1, NULL),
    (2, 2, 1, 125000.00, 125000.00, 5, 'RUB', 1, NULL),
    (3, 3, 3, 76000.00, 228000.00, 0, 'RUB', 1, NULL),
    (4, 4, 2, 98000.00, 196000.00, 0, 'RUB', 1, NULL),
    (5, 5, 1, 145000.00, 145000.00, 0, 'RUB', 1, NULL),
    (6, 6, 2, 168000.00, 336000.00, 10, 'RUB', 1, NULL),
    (7, 7, 1, 112000.00, 112000.00, 0, 'RUB', 1, NULL),
    (8, 8, 2, 155000.00, 310000.00, 0, 'RUB', 1, NULL),
    (9, 9, 1, 135000.00, 135000.00, 0, 'RUB', 1, NULL),
    (10, 10, 4, 45000.00, 180000.00, 0, 'RUB', 1, 'Группа'),
    (11, 1, 1, 89000.00, 89000.00, 0, 'RUB', 1, NULL),
    (12, 3, 2, 76000.00, 152000.00, 0, 'RUB', 1, NULL),
    (1, 2, 1, 125000.00, 125000.00, 0, 'RUB', 2, 'Доп. тур в той же брони'),
    (8, 7, 1, 112000.00, 112000.00, 0, 'RUB', 2, 'Комбо Италия');

INSERT INTO app.payment (
    booking_id, status_id, amount, paid_at, card_last4, card_token,
    payer_last_name, payer_first_name, payer_middle_name, payment_method, currency_code, transaction_ref
) VALUES
    (1, 3, 178000.00, now() - interval '10 days', '4242', encode(digest('tok-1', 'sha256'), 'hex'), 'Иванов', 'Иван', 'Иванович', 'card', 'RUB', 'TX-1001'),
    (2, 2, 125000.00, now() - interval '8 days', '1111', encode(digest('tok-2', 'sha256'), 'hex'), 'Петрова', 'Анна', 'Сергеевна', 'card', 'RUB', 'TX-1002'),
    (3, 3, 100000.00, now() - interval '7 days', '5555', encode(digest('tok-3a', 'sha256'), 'hex'), 'Сидоров', 'Пётр', 'Алексеевич', 'card', 'RUB', 'TX-1003'),
    (3, 3, 128000.00, now() - interval '6 days', '5555', encode(digest('tok-3b', 'sha256'), 'hex'), 'Сидоров', 'Пётр', 'Алексеевич', 'card', 'RUB', 'TX-1004'),
    (4, 2, 196000.00, now() - interval '5 days', '9999', encode(digest('tok-4', 'sha256'), 'hex'), 'Козлова', 'Мария', 'Игоревна', 'card', 'RUB', 'TX-1005'),
    (6, 3, 336000.00, now() - interval '4 days', '2222', encode(digest('tok-6', 'sha256'), 'hex'), 'Морозова', 'Елена', 'Викторовна', 'card', 'RUB', 'TX-1006'),
    (7, 2, 112000.00, now() - interval '3 days', '3333', encode(digest('tok-7', 'sha256'), 'hex'), 'Волков', 'Андрей', 'Николаевич', 'sbp', 'RUB', 'TX-1007'),
    (8, 3, 310000.00, now() - interval '2 days', '4444', encode(digest('tok-8', 'sha256'), 'hex'), 'Лебедева', 'Ольга', 'Дмитриевна', 'card', 'RUB', 'TX-1008'),
    (9, 3, 135000.00, now() - interval '1 days', '6666', encode(digest('tok-9', 'sha256'), 'hex'), 'Соколов', 'Максим', 'Павлович', 'card', 'RUB', 'TX-1009'),
    (10, 1, 180000.00, NULL, NULL, NULL, 'Зайцева', 'Наталья', 'Романовна', 'invoice', 'RUB', NULL),
    (12, 3, 152000.00, now() - interval '20 days', '7777', encode(digest('tok-12', 'sha256'), 'hex'), 'Павлова', 'Дарья', 'Андреевна', 'card', 'RUB', 'TX-1011'),
    (11, 5, 89000.00, now() - interval '15 days', '8888', encode(digest('tok-11', 'sha256'), 'hex'), 'Орлов', 'Кирилл', 'Евгеньевич', 'card', 'RUB', 'TX-1012');

INSERT INTO stg.client_import (
    last_name, first_name, middle_name, email, phone,
    passport_series, passport_number, birth_date, processed
) VALUES
    ('Иванов', 'Матвей', 'Иванович', 'testov@example.com', '+79009990001', '4599', '111111', '1990-01-01', false),
    ('Сидоров', 'Дмитрий', NULL, 'demo@example.com', '+79009990002', '4599', '222222', '1991-02-02', false);

-- привилегии
REVOKE ALL ON ALL TABLES IN SCHEMA app, ref, audit, stg FROM PUBLIC;
REVOKE ALL ON ALL SEQUENCES IN SCHEMA app, ref, audit, stg FROM PUBLIC;
REVOKE ALL ON ALL FUNCTIONS IN SCHEMA app, ref, audit, stg FROM PUBLIC;

GRANT USAGE ON SCHEMA ref TO app_reader, app_writer, app_owner, dml_admin, ddl_admin;
GRANT USAGE ON SCHEMA app TO app_reader, app_writer, app_owner, dml_admin, ddl_admin;
GRANT USAGE ON SCHEMA stg TO
    app_reader, app_writer, app_owner, auditor, ddl_admin, dml_admin, security_admin,
    login_reader, login_writer, login_owner, login_auditor,
    login_ddl, login_dml, login_sec;
GRANT USAGE ON SCHEMA audit TO auditor;
GRANT CREATE ON SCHEMA app, ref, audit, stg TO ddl_admin;
GRANT SELECT ON ALL TABLES IN SCHEMA ref TO app_reader, app_writer, app_owner, dml_admin;
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA ref TO dml_admin, app_owner;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA ref TO dml_admin, app_owner;
GRANT SELECT ON ALL TABLES IN SCHEMA app TO app_reader, app_writer, app_owner, dml_admin;
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO app_writer, app_owner, dml_admin;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA app TO app_writer, app_owner, dml_admin;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA stg TO app_writer, app_owner, dml_admin;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA stg TO app_writer, app_owner, dml_admin;
GRANT INSERT ON TABLE stg.session_start TO
    app_reader, app_writer, app_owner, auditor, ddl_admin, dml_admin, security_admin,
    login_reader, login_writer, login_owner, login_auditor,
    login_ddl, login_dml, login_sec;
GRANT USAGE, SELECT ON SEQUENCE stg.session_start_start_id_seq TO
    app_reader, app_writer, app_owner, auditor, ddl_admin, dml_admin, security_admin,
    login_reader, login_writer, login_owner, login_auditor,
    login_ddl, login_dml, login_sec;
GRANT SELECT ON ALL TABLES IN SCHEMA audit TO auditor;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA audit TO auditor;
ALTER DEFAULT PRIVILEGES REVOKE ALL ON TABLES FROM PUBLIC;
ALTER DEFAULT PRIVILEGES REVOKE ALL ON SEQUENCES FROM PUBLIC;
ALTER DEFAULT PRIVILEGES REVOKE ALL ON FUNCTIONS FROM PUBLIC;
ALTER DEFAULT PRIVILEGES IN SCHEMA app
GRANT SELECT ON TABLES TO app_reader, app_writer, app_owner, dml_admin;
ALTER DEFAULT PRIVILEGES IN SCHEMA app
GRANT INSERT, UPDATE, DELETE ON TABLES TO app_writer, app_owner, dml_admin;
ALTER DEFAULT PRIVILEGES IN SCHEMA app
GRANT USAGE, SELECT ON SEQUENCES TO app_writer, app_owner, dml_admin;
ALTER DEFAULT PRIVILEGES IN SCHEMA ref
GRANT SELECT ON TABLES TO app_reader, app_writer, app_owner, dml_admin;
ALTER DEFAULT PRIVILEGES IN SCHEMA ref
GRANT INSERT, UPDATE, DELETE ON TABLES TO dml_admin, app_owner;
ALTER DEFAULT PRIVILEGES IN SCHEMA audit
GRANT SELECT ON TABLES TO auditor;
ALTER DEFAULT PRIVILEGES IN SCHEMA stg
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_writer, app_owner, dml_admin;
DO $$
DECLARE
    r record;
BEGIN
FOR r IN
SELECT tablename
FROM pg_tables
WHERE schemaname = 'app'
LOOP
EXECUTE format('ALTER TABLE app.%I OWNER TO app_owner', r.tablename);
END LOOP;
FOR r IN
SELECT sequence_name
FROM information_schema.sequences
WHERE sequence_schema = 'app'
LOOP
EXECUTE format('ALTER SEQUENCE app.%I OWNER TO app_owner', r.sequence_name);
END LOOP;
END
$$;
GRANT CONNECT ON DATABASE travel_agency TO
    login_reader, login_writer, login_owner, login_auditor,
    login_ddl, login_dml, login_sec;

-- роли
DO $$
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'app_reader') THEN
        CREATE ROLE app_reader NOLOGIN;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'app_writer') THEN
        CREATE ROLE app_writer NOLOGIN;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'app_owner') THEN
        CREATE ROLE app_owner NOLOGIN;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'auditor') THEN
        CREATE ROLE auditor NOLOGIN;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'ddl_admin') THEN
        CREATE ROLE ddl_admin NOLOGIN;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'dml_admin') THEN
        CREATE ROLE dml_admin NOLOGIN;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'security_admin') THEN
        CREATE ROLE security_admin NOLOGIN;
    END IF;
END
$$;
DO $$
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'login_reader') THEN
        CREATE ROLE login_reader LOGIN PASSWORD 'reader' NOINHERIT;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'login_writer') THEN
        CREATE ROLE login_writer LOGIN PASSWORD 'writer' NOINHERIT;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'login_owner') THEN
        CREATE ROLE login_owner LOGIN PASSWORD 'owner' NOINHERIT;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'login_auditor') THEN
        CREATE ROLE login_auditor LOGIN PASSWORD 'auditor' NOINHERIT;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'login_ddl') THEN
        CREATE ROLE login_ddl LOGIN PASSWORD 'ddl' NOINHERIT;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'login_dml') THEN
        CREATE ROLE login_dml LOGIN PASSWORD 'dml' NOINHERIT;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'login_sec') THEN
        CREATE ROLE login_sec LOGIN PASSWORD 'sec' NOINHERIT;
    END IF;
END
$$;
GRANT app_reader TO login_reader;
GRANT app_writer TO login_writer;
GRANT app_owner TO login_owner;
GRANT auditor TO login_auditor;
GRANT ddl_admin TO login_ddl;
GRANT dml_admin TO login_dml;
GRANT security_admin TO login_sec;
GRANT app_reader TO app_writer;
GRANT app_writer TO app_owner;
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
INSERT INTO stg.session_start (application) VALUES ('тест записи входа у app_reader');
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
SELECT count(*) AS countries FROM ref.country;
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
SELECT count(*) AS clients FROM app.client;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
INSERT INTO stg.session_start (application) VALUES ('вставки у app_writer');
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
SELECT count(*) AS bookings FROM app.booking;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
INSERT INTO stg.client_import (
    last_name, first_name, email, phone, passport_series, passport_number, birth_date
) VALUES (
    'Временный', 'Клиент', 'temp@example.com', '+79001234567', '4510', '999001', '1999-01-01'
);
RESET ROLE;
SET ROLE login_owner;
SET ROLE app_owner;
INSERT INTO stg.session_start (application) VALUES ('вставка у app_owner');
RESET ROLE;
SET ROLE login_owner;
SET ROLE app_owner;
SELECT count(*) AS partners FROM app.partner;
RESET ROLE;
SET ROLE login_owner;
SET ROLE app_owner;
INSERT INTO ref.country (code, name) VALUES ('ZZ', 'Тест страны');
RESET ROLE;
SET ROLE login_owner;
SET ROLE app_owner;
DELETE FROM ref.country WHERE code = 'ZZ';
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
INSERT INTO stg.session_start (application) VALUES ('вставка у auditor');
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
SELECT login_id, login_time, username, client_ip, application
FROM audit.login_log
ORDER BY login_id DESC
RESET ROLE;
SET ROLE login_ddl;
SET ROLE ddl_admin;
INSERT INTO stg.session_start (application) VALUES ('вставка у ddl_admin');
RESET ROLE;
SET ROLE login_ddl;
SET ROLE ddl_admin;
CREATE TABLE IF NOT EXISTS app._ddl_test (id integer PRIMARY KEY);
RESET ROLE;
SET ROLE login_ddl;
SET ROLE ddl_admin;
DROP TABLE IF EXISTS app._ddl_test;
RESET ROLE;
SET ROLE login_dml;
SET ROLE dml_admin;
INSERT INTO stg.session_start (application) VALUES ('вставка у dml_admin');
RESET ROLE;
SET ROLE login_dml;
SET ROLE dml_admin;
SELECT count(*) AS tours FROM app.tour;
RESET ROLE;
SET ROLE login_dml;
SET ROLE dml_admin;
INSERT INTO ref.booking_status (code, name) VALUES ('test', 'Проверка');
RESET ROLE;
SET ROLE login_dml;
SET ROLE dml_admin;
DELETE FROM ref.booking_status WHERE code = 'test';
RESET ROLE;
SET ROLE login_sec;
SET ROLE security_admin;
INSERT INTO stg.session_start (application) VALUES ('вставка у security_admin');
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
INSERT INTO app.client (
    last_name, first_name, email, phone, passport_series, passport_number, birth_date
) VALUES (
    'тест', 'вставки', 'hack@example.com', '+70001112233', '0000', '000000', '2000-01-01'
);
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
DELETE FROM app.payment WHERE payment_id = 1;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
INSERT INTO audit.login_log (username) VALUES ('тест');
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
INSERT INTO ref.country (code, name) VALUES ('YY', 'тест страна');
RESET ROLE;
SET ROLE login_owner;
SET ROLE app_owner;
CREATE TABLE app._test (id integer PRIMARY KEY);
RESET ROLE;
SET ROLE login_owner;
SET ROLE app_owner;
SELECT login_id FROM audit.login_log LIMIT 1;
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
SELECT client_id, email FROM app.client LIMIT 1;
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
SELECT code FROM ref.country LIMIT 1;
RESET ROLE;
SET ROLE login_ddl;
SET ROLE ddl_admin;
SELECT count(*) FROM app.tour;
RESET ROLE;
SET ROLE login_dml;
SET ROLE dml_admin;
CREATE TABLE app.test (id integer PRIMARY KEY);
RESET ROLE;
SET ROLE login_dml;
SET ROLE dml_admin;
INSERT INTO audit.login_log (username) VALUES ('тест dml_admin');
RESET ROLE;
SET ROLE login_sec;
SET ROLE security_admin;
SELECT count(*) FROM app.tour;
RESET ROLE;
SET ROLE login_sec;
SET ROLE security_admin;
SELECT login_id FROM audit.login_log LIMIT 1;
RESET ROLE;
SET ROLE login_reader;
SELECT count(*) FROM app.tour;
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
SELECT count(*) FROM app.tour;

-- журнал входа
CREATE OR REPLACE FUNCTION audit.trg_session_start_login()
RETURNS trigger
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = app, public
AS
$$
BEGIN
    INSERT INTO audit.login_log (
        login_time,
        username,
        client_ip,
        session_role,
        application
    )
    VALUES (
        COALESCE(NEW.started_at, now()),
        NEW.actor,
        inet_client_addr(),
        session_user,
        NULLIF(NEW.application, '')
    );
	RETURN NEW;
END;
$$;
CREATE TRIGGER trg_session_start_login
    AFTER INSERT ON stg.session_start
    FOR EACH ROW
    EXECUTE FUNCTION audit.trg_session_start_login();
REVOKE ALL ON FUNCTION audit.trg_session_start_login() FROM PUBLIC;
ALTER FUNCTION audit.trg_session_start_login() OWNER TO postgres;

-- лр2

-- функции security definer
CREATE FUNCTION app.register_client(
    p_last_name text,
    p_first_name text,
    p_middle_name text,
    p_email text,
    p_phone text,
    p_passport_series text,
    p_passport_number text,
    p_birth_date date,
    p_citizenship text DEFAULT 'RU'
)
RETURNS integer
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = app, public
AS $$
DECLARE
    v_caller text;
    v_params jsonb;
    v_id integer;
BEGIN
    v_caller := NULLIF(current_setting('role', true), 'none');
    IF v_caller IS NULL THEN
        v_caller := session_user;
    END IF;
    v_params := jsonb_build_object(
        'last_name', p_last_name,
        'first_name', p_first_name,
        'email', p_email,
        'passport_series', p_passport_series,
        'passport_number', p_passport_number,
        'birth_date', p_birth_date,
        'citizenship', p_citizenship
    );
    BEGIN
        IF NULLIF(btrim(p_last_name), '') IS NULL OR NULLIF(btrim(p_first_name), '') IS NULL THEN
            RAISE EXCEPTION 'Фамилия и имя клиента обязательны';
        END IF;
        IF p_email IS NULL OR p_email !~* '^[^@]+@[^@]+\.[^@]+$' THEN
            RAISE EXCEPTION 'Некорректный адрес электронной почты';
        END IF;
        IF p_passport_series IS NULL OR p_passport_series !~ '^\d{4}$' THEN
            RAISE EXCEPTION 'Серия паспорта должна состоять из 4 цифр';
        END IF;
        IF p_passport_number IS NULL OR p_passport_number !~ '^\d{6}$' THEN
            RAISE EXCEPTION 'Номер паспорта должен состоять из 6 цифр';
        END IF;
        IF p_birth_date IS NULL OR p_birth_date >= CURRENT_DATE THEN
            RAISE EXCEPTION 'Дата рождения должна быть раньше текущей даты';
        END IF;
        IF p_citizenship IS NULL OR p_citizenship !~ '^[A-Z]{2}$' THEN
            RAISE EXCEPTION 'Гражданство задаётся двумя заглавными латинскими буквами';
        END IF;
        IF NULLIF(btrim(p_phone), '') IS NULL THEN
            RAISE EXCEPTION 'Телефон клиента обязателен';
        END IF;
        INSERT INTO app.client (
            last_name, first_name, middle_name, email, phone,
            passport_series, passport_number, birth_date, citizenship
        )
        VALUES (
            btrim(p_last_name), btrim(p_first_name), NULLIF(btrim(p_middle_name), ''),
            p_email, btrim(p_phone),
            p_passport_series, p_passport_number, p_birth_date, p_citizenship
        )
        RETURNING client_id INTO v_id;
        INSERT INTO audit.function_calls (function_name, caller_role, input_params, success)
        VALUES ('app.register_client', v_caller, v_params, true);
        RETURN v_id;
    EXCEPTION
        WHEN unique_violation THEN
            INSERT INTO audit.function_calls (function_name, caller_role, input_params, success)
            VALUES ('app.register_client', v_caller, v_params, false);
            RAISE EXCEPTION 'Клиент с такой почтой или паспортом уже зарегистрирован';
        WHEN OTHERS THEN
            INSERT INTO audit.function_calls (function_name, caller_role, input_params, success)
            VALUES ('app.register_client', v_caller, v_params, false);
            RAISE;
    END;
END;
$$;
SELECT app.register_client(
CREATE FUNCTION app.register_payment(
    p_booking_id integer,
    p_amount numeric,
    p_payer_last_name text,
    p_payer_first_name text,
    p_payer_middle_name text DEFAULT NULL,
    p_card_last4 text DEFAULT NULL,
    p_payment_method text DEFAULT 'card'
)
RETURNS integer
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = app, public
AS $$
DECLARE
    v_caller text;
    v_params jsonb;
    v_total numeric(12, 2);
    v_paid numeric(12, 2);
    v_status_id smallint;
    v_id integer;
    v_ref text;
BEGIN
    v_caller := NULLIF(current_setting('role', true), 'none');
    IF v_caller IS NULL THEN
        v_caller := session_user;
    END IF;
    v_params := jsonb_build_object(
        'booking_id', p_booking_id,
        'amount', p_amount,
        'payer_last_name', p_payer_last_name,
        'payer_first_name', p_payer_first_name,
        'card_last4', p_card_last4,
        'payment_method', p_payment_method
    );

    BEGIN
        IF p_booking_id IS NULL THEN
            RAISE EXCEPTION 'Не указана бронь';
        END IF;
        IF p_amount IS NULL OR p_amount <= 0 THEN
            RAISE EXCEPTION 'Сумма платежа должна быть больше нуля';
        END IF;
        IF NULLIF(btrim(p_payer_last_name), '') IS NULL
           OR NULLIF(btrim(p_payer_first_name), '') IS NULL THEN
            RAISE EXCEPTION 'Фамилия и имя плательщика обязательны';
        END IF;
        IF p_card_last4 IS NOT NULL AND p_card_last4 !~ '^\d{4}$' THEN
            RAISE EXCEPTION 'Последние цифры карты должны состоять из 4 цифр';
        END IF;

        SELECT total_amount
        INTO v_total
        FROM app.booking
        WHERE booking_id = p_booking_id;
        IF v_total IS NULL THEN
            RAISE EXCEPTION 'Бронь % не найдена', p_booking_id;
        END IF;

        SELECT COALESCE(SUM(amount), 0)
        INTO v_paid
        FROM app.payment
        WHERE booking_id = p_booking_id;
        IF v_paid + p_amount > v_total THEN
            RAISE EXCEPTION 'Сумма платежей (%) превышает сумму брони (%)',
                v_paid + p_amount, v_total;
        END IF;

        SELECT status_id
        INTO v_status_id
        FROM ref.payment_status
        WHERE code = 'captured';

        v_ref := 'LR2-' || gen_random_uuid()::text;

        INSERT INTO app.payment (
            booking_id, status_id, amount, paid_at, card_last4, card_token,
            payer_last_name, payer_first_name, payer_middle_name,
            payment_method, currency_code, transaction_ref
        )
        VALUES (
            p_booking_id, v_status_id, p_amount, now(), p_card_last4,
            encode(digest(v_ref, 'sha256'), 'hex'),
            btrim(p_payer_last_name), btrim(p_payer_first_name),
            NULLIF(btrim(p_payer_middle_name), ''),
            COALESCE(NULLIF(btrim(p_payment_method), ''), 'card'),
            'RUB', v_ref
        )
        RETURNING payment_id INTO v_id;

        INSERT INTO audit.function_calls (function_name, caller_role, input_params, success)
        VALUES ('app.register_payment', v_caller, v_params, true);
        RETURN v_id;
    EXCEPTION
        WHEN OTHERS THEN
            INSERT INTO audit.function_calls (function_name, caller_role, input_params, success)
            VALUES ('app.register_payment', v_caller, v_params, false);
            RAISE;
    END;
END;
$$;
SELECT app.register_payment(
CREATE FUNCTION app.set_booking_status(
    p_booking_id integer,
    p_new_code text
)
RETURNS integer
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = app, public
AS $$
DECLARE
    v_caller text;
    v_params jsonb;
    v_code text;
    v_new_id smallint;
BEGIN
    v_caller := NULLIF(current_setting('role', true), 'none');
    IF v_caller IS NULL THEN
        v_caller := session_user;
    END IF;
    v_params := jsonb_build_object(
        'booking_id', p_booking_id,
        'new_code', p_new_code
    );

    BEGIN
        IF p_booking_id IS NULL OR NULLIF(btrim(p_new_code), '') IS NULL THEN
            RAISE EXCEPTION 'Нужны идентификатор брони и код нового статуса';
        END IF;

        SELECT s.code
        INTO v_code
        FROM app.booking b
        JOIN ref.booking_status s ON s.status_id = b.status_id
        WHERE b.booking_id = p_booking_id;
        IF v_code IS NULL THEN
            RAISE EXCEPTION 'Бронь % не найдена', p_booking_id;
        END IF;

        SELECT status_id
        INTO v_new_id
        FROM ref.booking_status
        WHERE code = p_new_code;
        IF v_new_id IS NULL THEN
            RAISE EXCEPTION 'Неизвестный статус брони: %', p_new_code;
        END IF;

        IF NOT (
            (v_code = 'draft' AND p_new_code IN ('confirmed', 'cancelled'))
            OR (v_code = 'confirmed' AND p_new_code IN ('paid', 'cancelled'))
            OR (v_code = 'paid' AND p_new_code = 'completed')
        ) THEN
            RAISE EXCEPTION 'Нельзя перевести бронь из статуса % в %', v_code, p_new_code;
        END IF;

        UPDATE app.booking
        SET status_id = v_new_id
        WHERE booking_id = p_booking_id;

        INSERT INTO audit.function_calls (function_name, caller_role, input_params, success)
        VALUES ('app.set_booking_status', v_caller, v_params, true);
        RETURN p_booking_id;
    EXCEPTION
        WHEN OTHERS THEN
            INSERT INTO audit.function_calls (function_name, caller_role, input_params, success)
            VALUES ('app.set_booking_status', v_caller, v_params, false);
            RAISE;
    END;
END;
$$;
SELECT app.set_booking_status(
REVOKE ALL ON FUNCTION app.register_client(text, text, text, text, text, text, text, date, text) FROM PUBLIC;
REVOKE ALL ON FUNCTION app.register_payment(integer, numeric, text, text, text, text, text) FROM PUBLIC;
REVOKE ALL ON FUNCTION app.set_booking_status(integer, text) FROM PUBLIC;

ALTER FUNCTION app.register_client(text, text, text, text, text, text, text, date, text) OWNER TO postgres;
ALTER FUNCTION app.register_payment(integer, numeric, text, text, text, text, text) OWNER TO postgres;
ALTER FUNCTION app.set_booking_status(integer, text) OWNER TO postgres;

GRANT EXECUTE ON FUNCTION app.register_client(text, text, text, text, text, text, text, date, text)
    TO app_reader, app_writer, app_owner;
GRANT EXECUTE ON FUNCTION app.register_payment(integer, numeric, text, text, text, text, text)
    TO app_reader, app_writer, app_owner;
GRANT EXECUTE ON FUNCTION app.set_booking_status(integer, text)
    TO app_reader, app_writer, app_owner;
SET ROLE login_reader;
SET ROLE app_reader;
SELECT count(*) AS clients FROM app.client;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
SELECT app.register_payment(
	5, 1500.00, ‘Иванов’, ‘Иван’
) AS payment_id
SET ROLE login_reader;
SET ROLE app_reader;
	SELECT app.set_booking_status(5, 'confirmed') AS booking_id;
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
SELECT app.register_payment(
	5, 1000.00, ‘Иванов’, ‘Сергей’, ‘Сергеевич’, ‘6666’, ‘card’
) AS payment_id
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
SELECT app.register_client(
	‘Иванов’, ‘Иван’, ‘Иванович’,
	‘23re.13rew@mail.ru’, ‘+79000000001’,
	‘4510’, ‘100001’, DATE ‘1991-04-04’, ‘RU’
) AS client_id
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
SELECT call_id, call_time, function_name, caller_role, success, input_params FROM audit.function_calls
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
CREATE TABLE app.table (id integer PRIMARY KEY);
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
	INSERT INTO app.payment (
    	  booking_id, status_id, amount, payer_last_name, payer_first_name
) VALUES (
   	  5, 1, 10.00, 'Иванов', 'Иван'
);
	INSERT INTO audit.function_calls (function_name, caller_role, success)
VALUES ('forged', 'app_reader', true);
	RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
INSERT INTO audit.function_calls (function_name, caller_role, success)
VALUES ('значение', 'app_writer', true);
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
	SELECT app.register_payment(5, -5, 'Иванов', 'Иван', NULL, '6666', 'card');
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
	SELECT app.register_payment(11, 1.00, 'Иванов', 'Иван', NULL, '8888', 'card');
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
	SELECT app.register_client(
    'Иванов', 'Иван', NULL,
    '1234bad@mple.com', '+79000000002',
    '45', '100001', DATE '1991-04-04', 'RU'
);
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
	SELECT app.set_booking_status(11, 'confirmed');
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
	SELECT app.register_payment(5, 10, 'Иванов', 'Иван', NULL, NULL, 'card');
	RESET ROLE;
SET ROLE login_reader;
SELECT app.register_client(
    'Иванов', 'Иван', NULL,
    'l43.nrle@me.com', '+79000000003',
    '4511', '100002', DATE '1992-05-05', 'RU'
);

-- check и триггер
CREATE TABLE stg.bench_check (
    bench_id serial PRIMARY KEY,
    row_code varchar(32) NOT NULL,
    tour_id integer NOT NULL REFERENCES app.tour (tour_id),
    start_date date NOT NULL,
    end_date date NOT NULL,
    seats integer NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now(),
    CHECK (end_date >= start_date)
);
CREATE TABLE stg.bench_trg (
    bench_id serial PRIMARY KEY,
    row_code varchar(32) NOT NULL,
    tour_id integer NOT NULL REFERENCES app.tour (tour_id),
    start_date date NOT NULL,
    end_date date NOT NULL,
    seats integer NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);
CREATE FUNCTION stg.bench_dates()
RETURNS trigger
LANGUAGE plpgsql
SET search_path = app, public
AS $$
BEGIN
    IF NEW.end_date < NEW.start_date THEN
        RAISE EXCEPTION 'Дата окончания раньше даты начала';
    END IF;
    RETURN NEW;
END;
$$;
CREATE TRIGGER bench_dates
    BEFORE INSERT OR UPDATE ON stg.bench_trg
    FOR EACH ROW
    EXECUTE FUNCTION stg.bench_dates();
DO $$
DECLARE
    t0 timestamptz;
    t1 timestamptz;
BEGIN
    t0 := clock_timestamp();
    INSERT INTO stg.bench_check (row_code, tour_id, start_date, end_date, seats)
    SELECT 'C-' || g::text,
           (SELECT tour_id FROM app.tour ORDER BY tour_id LIMIT 1),
           DATE '2026-01-01', DATE '2026-01-01' + 7,
           2
    FROM generate_series(1, 10000) AS g;
    t1 := clock_timestamp();
    RAISE NOTICE 'CHECK: %', t1 - t0;

    t0 := clock_timestamp();
    INSERT INTO stg.bench_trg (row_code, tour_id, start_date, end_date, seats)
    SELECT 'T-' || g::text,
           (SELECT tour_id FROM app.tour ORDER BY tour_id LIMIT 1),
           DATE '2026-01-01', DATE '2026-01-01' + 7,
           2
    FROM generate_series(1, 10000) AS g;
    t1 := clock_timestamp();
    RAISE NOTICE 'триггер: %', t1 - t0;
END
$$;
INSERT INTO stg.bench_check (row_code, tour_id, start_date, end_date, seats)
VALUES ('bad-check', 1, DATE '2026-05-10', DATE '2026-05-01', 2);
INSERT INTO stg.bench_trg (row_code, tour_id, start_date, end_date, seats)
VALUES ('bad-trg', 1, DATE '2026-05-10', DATE '2026-05-01', 2);

-- журнал вызовов функций
CREATE TABLE audit.function_calls (
    call_id bigserial PRIMARY KEY,
    call_time timestamptz NOT NULL DEFAULT now(),
    function_name text NOT NULL,
    caller_role text NOT NULL,
    input_params jsonb,
    success boolean NOT NULL
);
REVOKE ALL ON TABLE audit.function_calls FROM PUBLIC;
REVOKE ALL ON SEQUENCE audit.function_calls_call_id_seq FROM PUBLIC;
GRANT SELECT ON TABLE audit.function_calls TO auditor;
GRANT SELECT ON SEQUENCE audit.function_calls_call_id_seq TO auditor;

-- лр3

-- филиалы
CREATE TABLE ref.segment (
    segment_id serial PRIMARY KEY,
    code varchar(32) NOT NULL UNIQUE,
    name varchar(100) NOT NULL
)
INSERT INTO ref.segment (segment_id, code, name) VALUES
    (1, 'msk', 'Москва'),
    (2, 'spb', 'Санкт-Петербург'),
    (3, 'nsk', 'Новосибирск'),
    (4, 'kzn', 'Казань'),
    (5, 'sochi', 'Сочи')
CREATE TABLE ref.role_segment (
    role_name name NOT NULL,
    segment_id integer NOT NULL REFERENCES ref.segment (segment_id),
    PRIMARY KEY (role_name, segment_id)
)
INSERT INTO ref.role_segment (role_name, segment_id) VALUES
    ('app_reader', 1),
    ('app_writer', 2),
    ('app_owner', 1),
    ('app_owner', 2),
    ('app_owner', 3),
    ('app_owner', 4),
    ('app_owner', 5),
    ('dml_admin', 1),
    ('dml_admin', 2),
    ('dml_admin', 3),
    ('dml_admin', 4),
    ('dml_admin', 5)
ALTER TABLE app.partner ADD COLUMN segment_id integer;
ALTER TABLE app.partner_agreement ADD COLUMN segment_id integer;
ALTER TABLE app.route ADD COLUMN segment_id integer;
ALTER TABLE app.tour ADD COLUMN segment_id integer;
ALTER TABLE app.client ADD COLUMN segment_id integer;
ALTER TABLE app.booking ADD COLUMN segment_id integer;
ALTER TABLE app.booking_tour ADD COLUMN segment_id integer;
ALTER TABLE app.payment ADD COLUMN segment_id integer
UPDATE app.partner SET segment_id = CASE
    WHEN partner_id IN (1, 2) THEN 1
    WHEN partner_id IN (3, 4) THEN 2
    WHEN partner_id IN (5, 6) THEN 3
    WHEN partner_id IN (7, 8) THEN 4
    ELSE 5
END;

UPDATE app.partner_agreement pa
SET segment_id = p.segment_id
FROM app.partner p
WHERE pa.partner_id = p.partner_id;

UPDATE app.route SET segment_id = CASE
    WHEN route_id IN (1, 2) THEN 1
    WHEN route_id IN (3, 4) THEN 2
    WHEN route_id IN (5, 6) THEN 3
    WHEN route_id IN (7, 8) THEN 4
    ELSE 5
END;

UPDATE app.tour t
SET segment_id = p.segment_id
FROM app.partner p
WHERE t.partner_id = p.partner_id;

UPDATE app.client SET segment_id = CASE
    WHEN client_id IN (1, 2) THEN 1
    WHEN client_id IN (3, 4) THEN 2
    WHEN client_id IN (5, 6) THEN 3
    WHEN client_id IN (7, 8, 9) THEN 4
    ELSE 5
END;

UPDATE app.booking b
SET segment_id = c.segment_id
FROM app.client c
WHERE b.client_id = c.client_id;

UPDATE app.booking_tour bt
SET segment_id = b.segment_id
FROM app.booking b
WHERE bt.booking_id = b.booking_id;

UPDATE app.payment pay
SET segment_id = b.segment_id
FROM app.booking b
WHERE pay.booking_id = b.booking_id;
DO $$
BEGIN
    BEGIN
        ALTER TABLE app.partner
            ADD FOREIGN KEY (segment_id) REFERENCES ref.segment (segment_id);
    EXCEPTION WHEN duplicate_object THEN NULL;
    END;
    BEGIN
        ALTER TABLE app.partner_agreement
            ADD FOREIGN KEY (segment_id) REFERENCES ref.segment (segment_id);
    EXCEPTION WHEN duplicate_object THEN NULL;
    END;
    BEGIN
        ALTER TABLE app.route
            ADD FOREIGN KEY (segment_id) REFERENCES ref.segment (segment_id);
    EXCEPTION WHEN duplicate_object THEN NULL;
    END;
    BEGIN
        ALTER TABLE app.tour
            ADD FOREIGN KEY (segment_id) REFERENCES ref.segment (segment_id);
    EXCEPTION WHEN duplicate_object THEN NULL;
    END;
    BEGIN
        ALTER TABLE app.client
            ADD FOREIGN KEY (segment_id) REFERENCES ref.segment (segment_id);
    EXCEPTION WHEN duplicate_object THEN NULL;
    END;
    BEGIN
        ALTER TABLE app.booking
            ADD FOREIGN KEY (segment_id) REFERENCES ref.segment (segment_id);
    EXCEPTION WHEN duplicate_object THEN NULL;
    END;
    BEGIN
        ALTER TABLE app.booking_tour
            ADD FOREIGN KEY (segment_id) REFERENCES ref.segment (segment_id);
    EXCEPTION WHEN duplicate_object THEN NULL;
    END;
    BEGIN
        ALTER TABLE app.payment
            ADD FOREIGN KEY (segment_id) REFERENCES ref.segment (segment_id);
    EXCEPTION WHEN duplicate_object THEN NULL;
    END;
END
$$;
CREATE INDEX app_partner_segment_id_idx ON app.partner (segment_id, partner_id);
CREATE INDEX app_partner_agreement_segment_id_idx ON app.partner_agreement (segment_id, agreement_id);
CREATE INDEX app_route_segment_id_idx ON app.route (segment_id, route_id);
CREATE INDEX app_tour_segment_id_idx ON app.tour (segment_id, tour_id);
CREATE INDEX app_client_segment_id_idx ON app.client (segment_id, client_id);
CREATE INDEX app_booking_segment_id_idx ON app.booking (segment_id, booking_id);
CREATE INDEX app_booking_tour_segment_id_idx ON app.booking_tour (segment_id, booking_id, tour_id);
CREATE INDEX app_payment_segment_id_idx ON app.payment (segment_id, payment_id);
CREATE INDEX ref_role_segment_role_name_idx ON ref.role_segment (role_name, segment_id)

-- rls
ALTER TABLE app.partner ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.partner FORCE ROW LEVEL SECURITY;
ALTER TABLE app.partner_agreement ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.partner_agreement FORCE ROW LEVEL SECURITY;
ALTER TABLE app.route ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.route FORCE ROW LEVEL SECURITY;
ALTER TABLE app.tour ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.tour FORCE ROW LEVEL SECURITY;
ALTER TABLE app.client ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.client FORCE ROW LEVEL SECURITY;
ALTER TABLE app.booking ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.booking FORCE ROW LEVEL SECURITY;
ALTER TABLE app.booking_tour ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.booking_tour FORCE ROW LEVEL SECURITY;
ALTER TABLE app.payment ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.payment FORCE ROW LEVEL SECURITY
CREATE FUNCTION app.rls_segment_allowed(p_segment_id integer)
RETURNS boolean
LANGUAGE sql
STABLE
SET search_path = app, ref, public
AS $$
    SELECT
        current_user = 'auditor'
        OR (
            NULLIF(current_setting('app.segment_id', true), '') IS NOT NULL
            AND p_segment_id = NULLIF(current_setting('app.segment_id', true), '')::integer
        )
        OR (
            NULLIF(current_setting('app.segment_id', true), '') IS NULL
            AND EXISTS (
                SELECT 1
                FROM ref.role_segment rs
                WHERE rs.role_name = current_user
                  AND rs.segment_id = p_segment_id
            )
        );
$$;
CREATE POLICY client_select ON app.client
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY client_insert ON app.client
    FOR INSERT
    TO app_writer, app_owner, dml_admin
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY client_update ON app.client
    FOR UPDATE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY client_delete ON app.client
    FOR DELETE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id));
GRANT SELECT ON TABLE ref.segment TO app_reader, app_writer, app_owner, dml_admin, auditor;
GRANT SELECT ON TABLE ref.role_segment TO app_reader, app_writer, app_owner, dml_admin, auditor

-- контекст сессии
CREATE FUNCTION app.set_session_ctx(p_segment_id integer, p_actor_id text)
RETURNS void
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = app, ref, public
AS $$
DECLARE
    v_role text;
    v_ok boolean;
BEGIN
    IF p_segment_id IS NULL THEN
        RAISE EXCEPTION 'Не указан segment_id';
    END IF;

    IF NOT EXISTS (SELECT 1 FROM ref.segment s WHERE s.segment_id = p_segment_id) THEN
        RAISE EXCEPTION 'Сегмент % не найден', p_segment_id;
    END IF;

    v_role := NULLIF(current_setting('role', true), 'none');
    IF v_role IS NULL THEN
        v_role := session_user;
    END IF;

    SELECT EXISTS (
        SELECT 1
        FROM ref.role_segment rs
        WHERE rs.role_name = v_role
          AND rs.segment_id = p_segment_id
    )
    INTO v_ok;

    IF NOT v_ok THEN
        RAISE EXCEPTION 'Роли % запрещён сегмент %', v_role, p_segment_id;
    END IF;

    PERFORM set_config('app.segment_id', p_segment_id::text, true);
    PERFORM set_config(
        'app.actor_id',
        COALESCE(NULLIF(btrim(p_actor_id), ''), v_role),
        true
    );
END;
$$;
ROLLBACK; --после предыдущих действий
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
BEGIN;
SELECT app.set_session_ctx(1, 'test');
SELECT count(*) FROM app.client;
COMMIT
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
SELECT count(*) AS reader_clients FROM app.client;
SELECT client_id, last_name, segment_id
FROM app.client
ORDER BY client_id
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
SELECT client_id, last_name, segment_id
FROM app.client
WHERE client_id = 7
ROLLBACK;
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
BEGIN;
SELECT app.set_session_ctx(1, 'app reader');
SELECT current_setting('app.segment_id', true) AS guc_segment,
       current_setting('app.actor_id', true) AS guc_actor;
SELECT count(*) AS reader_clients_ctx FROM app.client
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
BEGIN;
SELECT app.set_session_ctx(2, 'читатель')

RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
BEGIN;
SELECT app.set_session_ctx(3, 'читатель')
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'writer-demo');
SELECT count(*) AS writer_clients FROM app.client;
SELECT client_id, last_name, segment_id
FROM app.client
ORDER BY client_id
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'app writer');
SELECT client_id, last_name, segment_id
FROM app.client
WHERE client_id = 1
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(1, 'app_writer')
ROLLBACK;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'app writer');
INSERT INTO app.client (
    segment_id, last_name, first_name, email, phone,
    passport_series, passport_number, birth_date, citizenship
) VALUES (
    1, 'rsfh', 'seths', 'sethet@sthts.com', '+34567534566',
    '1111', '111111', DATE '1999-01-01', 'RU'
)
ROLLBACK;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'app writer');
INSERT INTO app.client (
    segment_id, last_name, first_name, email, phone,
    passport_series, passport_number, birth_date, citizenship
) VALUES (
    2, 'test', 'ethzgf', 'SFgsfg@srgrf.ru', '+790071123542',
    '1112', '222222', DATE '1991-02-02', 'RU'
)
RETURNING client_id, segment_id, email
ROLLBACK;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'app writer');
UPDATE app.client
SET segment_id = 1
WHERE email = 'vika@mail.ru'
ROLLBACK;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'app writer');
UPDATE app.client
SET phone = '+79000000999'
WHERE email = 'vika@mail.ru'
RETURNING client_id, segment_id, phone
ROLLBACK;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'app writer');
UPDATE app.client
SET phone = '+79000000000'
WHERE client_id = 1
ROLLBACK;
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'app writer');
DELETE FROM app.client
WHERE client_id = 1
RESET ROLE;
SET ROLE login_owner;
SET ROLE app_owner;
BEGIN;
SELECT app.set_session_ctx(3, 'app owner');
SELECT count(*) AS owner_nsk_clients FROM app.client;
SELECT client_id, last_name, segment_id
FROM app.client
ORDER BY client_id
RESET ROLE;
SET ROLE login_dml;
SET ROLE dml_admin;
BEGIN;
SELECT app.set_session_ctx(5, 'dml');
SELECT count(*) AS dml_sochi_clients FROM app.client;
SELECT client_id, last_name, segment_id
FROM app.client
ORDER BY client_id
RESET ROLE;
SET ROLE login_dml;
SET ROLE dml_admin;
BEGIN;
SELECT app.set_session_ctx(5, 'dml');
INSERT INTO app.client (
    segment_id, last_name, first_name, email, phone,
    passport_series, passport_number, birth_date, citizenship
) VALUES (
    1, 'dgh', 'dhd', 'q24re@sdth.ru', '+79003598999',
    '4545', '555555', DATE '1995-05-05', 'RU'
)
ROLLBACK;
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
INSERT INTO app.client (
    segment_id, last_name, first_name, email, phone,
    passport_series, passport_number, birth_date
) VALUES (
    1, 'ерыре', 'уерв', 'уыер@уеру.com', '+79124234004',
    '4449', '444444', DATE '1990-04-04'
)
GRANT USAGE ON SCHEMA app TO auditor;
GRANT USAGE ON SCHEMA ref TO auditor;
GRANT SELECT ON ALL TABLES IN SCHEMA app TO auditor;
GRANT SELECT ON ALL TABLES IN SCHEMA ref TO auditor
CREATE POLICY client_select ON app.client
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY payment_select ON app.payment
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_select ON app.booking
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_tour_select ON app.booking_tour
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY tour_select ON app.tour
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY partner_select ON app.partner
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY agreement_select ON app.partner_agreement
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY route_select ON app.route
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
SELECT count(*) AS auditor_clients FROM app.client;
SELECT segment_id, count(*) AS n
FROM app.client
GROUP BY segment_id
ORDER BY segment_id
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
SELECT client_id, last_name, segment_id FROM app.client WHERE client_id = 7
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
SELECT (SELECT count(*) FROM app.client) AS auditor_clients
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
INSERT INTO app.client (
    segment_id, last_name, first_name, email, phone,
    passport_series, passport_number, birth_date
) VALUES (
    1, 'regfr', 'ersygr', 'drtge@mail.ru', '+79000000003',
    '5476', '333333', DATE '1993-03-03'
)
RESET ROLE;
SET ROLE login_auditor;
SET ROLE auditor;
BEGIN;
SELECT app.set_session_ctx(1, 'auditor')
CREATE POLICY payment_select ON app.payment
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY payment_insert ON app.payment
    FOR INSERT
    TO app_writer, app_owner, dml_admin
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY payment_update ON app.payment
    FOR UPDATE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY payment_delete ON app.payment
    FOR DELETE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_select ON app.booking
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_insert ON app.booking
    FOR INSERT
    TO app_writer, app_owner, dml_admin
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_update ON app.booking
    FOR UPDATE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_delete ON app.booking
    FOR DELETE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_tour_select ON app.booking_tour
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_tour_insert ON app.booking_tour
    FOR INSERT
    TO app_writer, app_owner, dml_admin
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_tour_update ON app.booking_tour
    FOR UPDATE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY booking_tour_delete ON app.booking_tour
    FOR DELETE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY tour_select ON app.tour
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));
CREATE POLICY tour_insert ON app.tour
    FOR INSERT
    TO app_writer, app_owner, dml_admin
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY tour_update ON app.tour
    FOR UPDATE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY tour_delete ON app.tour
    FOR DELETE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY partner_select ON app.partner
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY partner_insert ON app.partner
    FOR INSERT
    TO app_writer, app_owner, dml_admin
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY partner_update ON app.partner
    FOR UPDATE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY partner_delete ON app.partner
    FOR DELETE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY agreement_select ON app.partner_agreement
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY agreement_insert ON app.partner_agreement
    FOR INSERT
    TO app_writer, app_owner, dml_admin
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY agreement_update ON app.partner_agreement
    FOR UPDATE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY agreement_delete ON app.partner_agreement
    FOR DELETE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY route_select ON app.route
    FOR SELECT
    TO app_reader, app_writer, app_owner, dml_admin, auditor
    USING (app.rls_segment_allowed(segment_id));

CREATE POLICY route_insert ON app.route
    FOR INSERT
    TO app_writer, app_owner, dml_admin
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY route_update ON app.route
    FOR UPDATE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))
    WITH CHECK (app.rls_segment_allowed(segment_id));

CREATE POLICY route_delete ON app.route
    FOR DELETE
    TO app_writer, app_owner, dml_admin
    USING (app.rls_segment_allowed(segment_id))

-- лр4

-- представления
CREATE VIEW app.v_client_work
WITH (security_barrier = true, security_invoker = true) AS
SELECT
    c.client_id,
    c.last_name,
    c.first_name,
    c.email,
    c.phone,
    c.citizenship,
    c.segment_id,
    c.created_at
FROM app.client c
WHERE c.citizenship = 'RU'
WITH CHECK OPTION.
WITH CHECK OPTION означает: любая строка, которую вставляют или меняют через это представление, после операции обязана снова удовлетворять WHERE представления. Поэтому, например, такой запрос отклоняется, так как после него строка уже не удовлетворяет условию WHERE c.citizenship = 'RU':
UPDATE app.v_client_work
SET citizenship = 'KZ'
WHERE client_id = 3.
CREATE VIEW app.v_booking_segment_stats
WITH (security_barrier = true, security_invoker = true) AS
SELECT
    b.segment_id,
    b.status_id,
    count(*)::bigint AS booking_cnt,
    coalesce(sum(b.total_amount), 0)::numeric(14, 2) AS amount_sum
FROM app.booking b
GROUP BY b.segment_id, b.status_id.
ALTER VIEW app.v_client_work OWNER TO app_owner;
ALTER VIEW app.v_booking_segment_stats OWNER TO app_owner;
REVOKE ALL ON TABLE app.v_client_work FROM PUBLIC;
REVOKE ALL ON TABLE app.v_booking_segment_stats FROM PUBLIC;
GRANT SELECT ON TABLE app.v_client_work TO app_reader, app_writer, app_owner, dml_admin, auditor;
GRANT INSERT, UPDATE, DELETE ON TABLE app.v_client_work TO app_writer, app_owner, dml_admin;
GRANT SELECT ON TABLE app.v_booking_segment_stats TO app_reader, app_writer, app_owner, dml_admin, auditor.

-- аудит изменений
CREATE TABLE audit.row_change_log (
    change_id bigserial PRIMARY KEY,
    changed_at timestamptz NOT NULL DEFAULT now(),
    actor_role text NOT NULL,
    session_login text NOT NULL,
    table_schema text NOT NULL,
    table_name text NOT NULL,
    operation text NOT NULL,
    row_pk text NOT NULL,
    old_data jsonb,
    new_data jsonb
)
	REVOKE ALL ON TABLE audit.row_change_log FROM PUBLIC;
GRANT SELECT ON TABLE audit.row_change_log TO auditor.
CREATE FUNCTION audit.mask_email(p_email text)
RETURNS text
LANGUAGE sql
IMMUTABLE
SET search_path = public
AS $$
    SELECT CASE
        WHEN p_email IS NULL OR position('@' IN p_email) = 0 THEN '***'
        ELSE left(p_email, 1) || '***@' || split_part(p_email, '@', 2)
    END;
$$.
CREATE FUNCTION audit.hash_secret(p_value text)
RETURNS text
LANGUAGE sql
IMMUTABLE
SET search_path = public
AS $$
    SELECT CASE
        WHEN p_value IS NULL THEN NULL
        ELSE encode(digest(p_value, 'sha256'), 'hex')
    END;
$$.
REVOKE ALL ON FUNCTION audit.mask_email(text) FROM PUBLIC;
REVOKE ALL ON FUNCTION audit.hash_secret(text) FROM PUBLIC;
ALTER FUNCTION audit.mask_email(text) OWNER TO postgres;
ALTER FUNCTION audit.hash_secret(text) OWNER TO postgres;
GRANT EXECUTE ON FUNCTION audit.mask_email(text) TO postgres;
GRANT EXECUTE ON FUNCTION audit.hash_secret(text) TO postgres.
CREATE FUNCTION audit.trg_client_row_change()
RETURNS trigger
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = audit, app, public
AS $$
DECLARE
    v_actor text;
    v_old jsonb;
    v_new jsonb;
BEGIN
    v_actor := current_user;

    IF TG_OP = 'UPDATE' OR TG_OP = 'DELETE' THEN
        v_old := jsonb_build_object(
            'client_id', OLD.client_id,
            'last_name', OLD.last_name,
            'first_name', OLD.first_name,
            'middle_name', OLD.middle_name,
            'email', audit.mask_email(OLD.email),
            'phone', left(COALESCE(OLD.phone, ''), 4) || '****',
            'passport_series_hash', audit.hash_secret(OLD.passport_series),
            'passport_number_hash', audit.hash_secret(OLD.passport_number),
            'birth_date', OLD.birth_date,
            'citizenship', OLD.citizenship,
            'segment_id', OLD.segment_id,
            'created_at', OLD.created_at
        );
    END IF;

    IF TG_OP = 'UPDATE' THEN
        v_new := jsonb_build_object(
            'client_id', NEW.client_id,
            'last_name', NEW.last_name,
            'first_name', NEW.first_name,
            'middle_name', NEW.middle_name,
            'email', audit.mask_email(NEW.email),
            'phone', left(COALESCE(NEW.phone, ''), 4) || '****',
            'passport_series_hash', audit.hash_secret(NEW.passport_series),
            'passport_number_hash', audit.hash_secret(NEW.passport_number),
            'birth_date', NEW.birth_date,
            'citizenship', NEW.citizenship,
            'segment_id', NEW.segment_id,
            'created_at', NEW.created_at
        );

        INSERT INTO audit.row_change_log (
            actor_role, session_login, table_schema, table_name,
            operation, row_pk, old_data, new_data
        ) VALUES (
            v_actor, session_user, TG_TABLE_SCHEMA, TG_TABLE_NAME,
            TG_OP, OLD.client_id::text, v_old, v_new
        );
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit.row_change_log (
            actor_role, session_login, table_schema, table_name,
            operation, row_pk, old_data, new_data
        ) VALUES (
            v_actor, session_user, TG_TABLE_SCHEMA, TG_TABLE_NAME,
            TG_OP, OLD.client_id::text, v_old, NULL
        );
        RETURN OLD;
    END IF;

    RETURN NULL;
END;
$$;
CREATE TRIGGER trg_client_row_change
    AFTER UPDATE OR DELETE ON app.client
    FOR EACH ROW
    EXECUTE FUNCTION audit.trg_client_row_change().
CREATE FUNCTION audit.trg_booking_row_change()
RETURNS trigger
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = audit, app, public
AS $$
DECLARE
    v_actor text;
    v_old jsonb;
    v_new jsonb;
BEGIN
    v_actor := current_user;

    IF TG_OP = 'UPDATE' OR TG_OP = 'DELETE' THEN
        v_old := jsonb_build_object(
            'booking_id', OLD.booking_id,
            'client_id', OLD.client_id,
            'status_id', OLD.status_id,
            'booked_at', OLD.booked_at,
            'total_amount', OLD.total_amount,
            'discount_percent', OLD.discount_percent,
            'manager_note', OLD.manager_note,
            'confirmation_code', OLD.confirmation_code,
            'source_channel', OLD.source_channel,
            'segment_id', OLD.segment_id
        );
    END IF;

    IF TG_OP = 'UPDATE' THEN
        v_new := jsonb_build_object(
            'booking_id', NEW.booking_id,
            'client_id', NEW.client_id,
            'status_id', NEW.status_id,
            'booked_at', NEW.booked_at,
            'total_amount', NEW.total_amount,
            'discount_percent', NEW.discount_percent,
            'manager_note', NEW.manager_note,
            'confirmation_code', NEW.confirmation_code,
            'source_channel', NEW.source_channel,
            'segment_id', NEW.segment_id
        );

        INSERT INTO audit.row_change_log (
            actor_role, session_login, table_schema, table_name,
            operation, row_pk, old_data, new_data
        ) VALUES (
            v_actor, session_user, TG_TABLE_SCHEMA, TG_TABLE_NAME,
            TG_OP, OLD.booking_id::text, v_old, v_new
        );
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit.row_change_log (
            actor_role, session_login, table_schema, table_name,
            operation, row_pk, old_data, new_data
        ) VALUES (
            v_actor, session_user, TG_TABLE_SCHEMA, TG_TABLE_NAME,
            TG_OP, OLD.booking_id::text, v_old, NULL
        );
        RETURN OLD;
    END IF;

    RETURN NULL;
END;
$$;
CREATE TRIGGER trg_booking_row_change
    AFTER UPDATE OR DELETE ON app.booking
    FOR EACH ROW
    EXECUTE FUNCTION audit.trg_booking_row_change().
CREATE FUNCTION audit.trg_payment_row_change()
RETURNS trigger
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = audit, app, public
AS $$
DECLARE
    v_actor text;
    v_old jsonb;
    v_new jsonb;
BEGIN
    v_actor := current_user;

    IF TG_OP = 'UPDATE' OR TG_OP = 'DELETE' THEN
        v_old := jsonb_build_object(
            'payment_id', OLD.payment_id,
            'booking_id', OLD.booking_id,
            'status_id', OLD.status_id,
            'amount', OLD.amount,
            'paid_at', OLD.paid_at,
            'card_last4', OLD.card_last4,
            'card_token_hash', audit.hash_secret(OLD.card_token),
            'payer_last_name', OLD.payer_last_name,
            'payer_first_name', OLD.payer_first_name,
            'payment_method', OLD.payment_method,
            'currency_code', OLD.currency_code,
            'transaction_ref', OLD.transaction_ref,
            'segment_id', OLD.segment_id
        );
    END IF;

    IF TG_OP = 'UPDATE' THEN
        v_new := jsonb_build_object(
            'payment_id', NEW.payment_id,
            'booking_id', NEW.booking_id,
            'status_id', NEW.status_id,
            'amount', NEW.amount,
            'paid_at', NEW.paid_at,
            'card_last4', NEW.card_last4,
            'card_token_hash', audit.hash_secret(NEW.card_token),
            'payer_last_name', NEW.payer_last_name,
            'payer_first_name', NEW.payer_first_name,
            'payment_method', NEW.payment_method,
            'currency_code', NEW.currency_code,
            'transaction_ref', NEW.transaction_ref,
            'segment_id', NEW.segment_id
        );

        INSERT INTO audit.row_change_log (
            actor_role, session_login, table_schema, table_name,
            operation, row_pk, old_data, new_data
        ) VALUES (
            v_actor, session_user, TG_TABLE_SCHEMA, TG_TABLE_NAME,
            TG_OP, OLD.payment_id::text, v_old, v_new
        );
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit.row_change_log (
            actor_role, session_login, table_schema, table_name,
            operation, row_pk, old_data, new_data
        ) VALUES (
            v_actor, session_user, TG_TABLE_SCHEMA, TG_TABLE_NAME,
            TG_OP, OLD.payment_id::text, v_old, NULL
        );
        RETURN OLD;
    END IF;

    RETURN NULL;
END;
$$;
CREATE TRIGGER trg_payment_row_change
    AFTER UPDATE OR DELETE ON app.payment
    FOR EACH ROW
    EXECUTE FUNCTION audit.trg_payment_row_change().
ALTER FUNCTION audit.trg_client_row_change() OWNER TO postgres;
ALTER FUNCTION audit.trg_booking_row_change() OWNER TO postgres;
ALTER FUNCTION audit.trg_payment_row_change() OWNER TO postgres;
REVOKE ALL ON FUNCTION audit.trg_client_row_change() FROM PUBLIC;
REVOKE ALL ON FUNCTION audit.trg_booking_row_change() FROM PUBLIC;
REVOKE ALL ON FUNCTION audit.trg_payment_row_change() FROM PUBLIC.
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'asdf');
UPDATE app.v_client_work
SET citizenship = 'KZ'
WHERE client_id = 3
RETURNING client_id, citizenship.
RESET ROLE;
SET ROLE login_reader;
SET ROLE app_reader;
BEGIN;
SELECT app.set_session_ctx(1, 'asdf');
SELECT segment_id, status_id, booking_cnt, amount_sum
FROM app.v_booking_segment_stats
ORDER BY segment_id, status_id.
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'asdfg');
UPDATE app.client
SET phone = '+71111111111'
WHERE client_id = 3
RETURNING client_id, phone, segment_id.
RESET ROLE;
SET ROLE login_writer;
SET ROLE app_writer;
BEGIN;
SELECT app.set_session_ctx(2, 'asdfgh');
DELETE FROM app.client
WHERE client_id = 1

-- индексы
CREATE INDEX app_booking_segment_status_idx ON app.booking (segment_id, status_id);
CREATE INDEX app_payment_segment_status_idx ON app.payment (segment_id, status_id);
CREATE INDEX app_client_segment_citizenship_idx ON app.client (segment_id, citizenship).

-- архив журнала
CREATE TABLE audit.row_change_log_archive (
    change_id bigint PRIMARY KEY,
    changed_at timestamptz NOT NULL,
    actor_role text NOT NULL,
    session_login text NOT NULL,
    table_schema text NOT NULL,
    table_name text NOT NULL,
    operation text NOT NULL,
    row_pk text NOT NULL,
    old_data jsonb,
    new_data jsonb,
    archived_at timestamptz NOT NULL DEFAULT now()
)
CREATE FUNCTION audit.backup_audit_logs(days_interval integer)
RETURNS integer
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = audit, public
AS $$
DECLARE
    v_moved integer := 0;
    v_cutoff timestamptz;
BEGIN
    IF days_interval IS NULL OR days_interval < 0 THEN
        RAISE EXCEPTION 'days_interval должен быть >= 0';
    END IF;

    v_cutoff := now() - make_interval(days => days_interval);

    WITH moved AS (
        DELETE FROM audit.row_change_log src
        WHERE src.changed_at < v_cutoff
        RETURNING
            src.change_id,
            src.changed_at,
            src.actor_role,
            src.session_login,
            src.table_schema,
            src.table_name,
            src.operation,
            src.row_pk,
            src.old_data,
            src.new_data
    )
    INSERT INTO audit.row_change_log_archive (
        change_id, changed_at, actor_role, session_login,
        table_schema, table_name, operation, row_pk, old_data, new_data
    )
    SELECT
        m.change_id,
        m.changed_at,
        m.actor_role,
        m.session_login,
        m.table_schema,
        m.table_name,
        m.operation,
        m.row_pk,
        m.old_data,
        m.new_data
    FROM moved m;

    GET DIAGNOSTICS v_moved = ROW_COUNT;
    RETURN v_moved;
END;
$$.
REVOKE ALL ON TABLE audit.row_change_log_archive FROM PUBLIC;
REVOKE ALL ON SEQUENCE audit.row_change_log_change_id_seq FROM PUBLIC;
GRANT SELECT ON TABLE audit.row_change_log_archive TO auditor;
GRANT SELECT ON SEQUENCE audit.row_change_log_change_id_seq TO auditor;
ALTER FUNCTION audit.backup_audit_logs(integer) OWNER TO postgres;
REVOKE ALL ON FUNCTION audit.backup_audit_logs(integer) FROM PUBLIC;
GRANT USAGE ON SCHEMA audit TO security_admin, dml_admin;
GRANT EXECUTE ON FUNCTION audit.backup_audit_logs(integer) TO security_admin, dml_admin.
UPDATE audit.row_change_log
SET changed_at = now() - interval '40 days'
WHERE change_id = 1.
RESET ROLE;
SELECT audit.backup_audit_logs(30) AS moved_rows.

-- временные привилегии

CREATE TABLE audit.temp_access_log (
    request_id bigserial PRIMARY KEY,
    request_time timestamptz NOT NULL DEFAULT clock_timestamp(),
    caller_role text NOT NULL,
    operation text NOT NULL,
    expires_at timestamptz NOT NULL
);

REVOKE ALL ON TABLE audit.temp_access_log FROM PUBLIC;
REVOKE ALL ON SEQUENCE audit.temp_access_log_request_id_seq FROM PUBLIC;
GRANT SELECT ON TABLE audit.temp_access_log TO auditor;
GRANT SELECT ON SEQUENCE audit.temp_access_log_request_id_seq TO auditor;

CREATE FUNCTION app.request_temp_privilege(operation_name text, duration_min integer)
RETURNS timestamptz
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = audit, pg_catalog
AS $$
DECLARE
    v_caller text;
    v_expires timestamptz;
BEGIN
    v_caller := COALESCE(
        NULLIF(NULLIF(current_setting('role', true), ''), 'none'),
        session_user::text
    );

    IF v_caller <> 'app_writer' OR operation_name IS DISTINCT FROM 'delete_booking' THEN
        RAISE EXCEPTION 'Операция % не разрешена роли %', operation_name, v_caller;
    END IF;
    IF duration_min IS NULL OR duration_min < 1 THEN
        RAISE EXCEPTION 'duration_min должен быть не меньше 1';
    END IF;

    v_expires := clock_timestamp() + make_interval(mins => duration_min);

    INSERT INTO audit.temp_access_log (caller_role, operation, expires_at)
    VALUES (v_caller, operation_name, v_expires);

    -- false: метка живёт до конца сессии, а не только до COMMIT
    PERFORM set_config('app.temp_privilege_until', v_expires::text, false);
    RETURN v_expires;
END;
$$;

CREATE FUNCTION app.delete_booking(p_booking_id integer)
RETURNS integer
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = app, ref, audit, pg_catalog
AS $$
DECLARE
    v_caller text;
    v_guc text;
    v_exp timestamptz;
    v_segment integer;
BEGIN
    v_caller := COALESCE(
        NULLIF(NULLIF(current_setting('role', true), ''), 'none'),
        session_user::text
    );
    v_guc := NULLIF(current_setting('app.temp_privilege_until', true), '');

    SELECT l.expires_at
    INTO v_exp
    FROM audit.temp_access_log l
    WHERE l.caller_role = v_caller
      AND l.operation = 'delete_booking'
    ORDER BY l.request_id DESC
    LIMIT 1;

    IF v_exp IS NULL
       OR v_guc IS NULL
       OR v_exp <= clock_timestamp()
       OR v_guc::timestamptz <= clock_timestamp()
    THEN
        RAISE EXCEPTION 'Временная привилегия delete_booking не активна или истекла';
    END IF;

    -- Владелец функции — postgres и обходит RLS, поэтому филиал проверяется здесь.
    SELECT b.segment_id
    INTO v_segment
    FROM app.booking b
    WHERE b.booking_id = p_booking_id;

    IF v_segment IS NULL
       OR NOT EXISTS (
            SELECT 1
            FROM ref.role_segment rs
            WHERE rs.role_name = v_caller
              AND rs.segment_id = v_segment
       )
    THEN
        RAISE EXCEPTION 'Бронь % не найдена или недоступна роли %', p_booking_id, v_caller;
    END IF;

    DELETE FROM app.payment WHERE booking_id = p_booking_id;
    DELETE FROM app.booking_tour WHERE booking_id = p_booking_id;
    DELETE FROM app.booking WHERE booking_id = p_booking_id;
    RETURN p_booking_id;
END;
$$;

REVOKE ALL ON FUNCTION app.request_temp_privilege(text, integer) FROM PUBLIC;
REVOKE ALL ON FUNCTION app.delete_booking(integer) FROM PUBLIC;
GRANT EXECUTE ON FUNCTION app.request_temp_privilege(text, integer) TO app_writer;
GRANT EXECUTE ON FUNCTION app.delete_booking(integer) TO app_writer;
