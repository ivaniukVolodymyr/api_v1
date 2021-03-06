
-- Stored procedures:

USE `sql11436082`;
DROP procedure IF EXISTS `create_meal`;
DELIMITER $$
USE `sql11436082`$$
CREATE DEFINER=`root`@`localhost` PROCEDURE `create_meal`(
    IN coachId BIGINT,
    IN name NVARCHAR(60),
    IN procedureText NVARCHAR(600),
    IN locale NVARCHAR(60),
    IN addToCoachMeals bit
)
BEGIN
   SET @id = IFNULL((SELECT MAX(id)+1 FROM sql11436082.meal),0);
   SET @addedDate = NOW();
	-- Create proper meal  
   INSERT IGNORE INTO sql11436082.meal(id, procedure_text, created_at)
   VALUES (@id, procedureText, @addedDate);

   INSERT IGNORE INTO sql11436082.meal_i18n(id, locale, name, created_at)
   VALUES (@id, locale, name, @addedDate);
   -- Create proper record for coach
   IF (addToCoachMeals = true) then
   INSERT IGNORE INTO sql11436082.coach_meal (coach_id, id, added_at)
   VALUES (coachId, @id, @addedDate);
   END IF;
   SELECT @id AS id;	
 END$$
DELIMITER ;

USE `sql11436082`;
DROP procedure IF EXISTS `delete_meal`;
DELIMITER $$
USE `sql11436082`$$
CREATE DEFINER=`root`@`localhost` PROCEDURE `delete_meal`(
    IN mealId BIGINT
)
BEGIN
   -- delete all related meal records
   DELETE FROM sql11436082.meal where id = mealId;
   DELETE FROM sql11436082.meal_i18n where id = mealId;
   DELETE FROM sql11436082.coach_meal where id = mealId;
   
 END$$
DELIMITER ;

-- Schema and tables creation:

CREATE SCHEMA IF NOT EXISTS sql11436082;

CREATE TABLE IF NOT EXISTS sql11436082.meal (
    id BIGINT NOT NULL,
    procedure_text NVARCHAR(1000) NOT NULL,
    created_at DATETIME,
    PRIMARY KEY (id ASC),
    INDEX m_procedure_text_IX (procedure_text ASC)
);

CREATE TABLE IF NOT EXISTS sql11436082.meal_i18n (
    id BIGINT NOT NULL,
    locale NVARCHAR(60) NOT NULL,
    name NVARCHAR(60) NOT NULL,
    created_at DATETIME,
    PRIMARY KEY (id ASC, locale ASC),
    INDEX m_name_IX (name ASC)
);

CREATE TABLE IF NOT EXISTS sql11436082.coach_meal (
    coach_id BIGINT NOT NULL,
    id BIGINT NOT NULL,
    added_at DATETIME,
    PRIMARY KEY (coach_id ASC, id ASC)
);


CREATE TABLE IF NOT EXISTS sql11436082.coach (
    coach_id BIGINT NOT NULL,
    full_name NVARCHAR(120) NOT NULL,
    created_at DATETIME,
    PRIMARY KEY (coach_id ASC),
    INDEX c_full_name_IX (full_name ASC)
);


-- queries can be found in files:
  \models\meals.js
  \models\locale.js