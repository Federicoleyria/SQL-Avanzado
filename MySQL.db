--Procesamiento almacenado para calcular una la utilidad desde una fecha hasta otra fecha
CREATE DEFINER=`root`@`localhost` FUNCTION `UtilidadTotal`(
	`_FechaDesde` DATE,
	`_FechaHasta` DATE
)
RETURNS decimal(10,0)
LANGUAGE SQL
NOT DETERMINISTIC
READS SQL DATA
SQL SECURITY DEFINER
COMMENT ''
BEGIN
	DECLARE Ganancia DECIMAL(10,2);
	
	SELECT SUM(VD_Cantidad *(VD_Precio - VD_Costo)) INTO Ganancia
		FROM Ventas_Detalle
		JOIN ventas ON VD_VentasID = Ventas_ID
	WHERE Ventas_Fecha BETWEEN _FechaDesde AND _FechaHasta AND 
	VD_Costo>0 AND VD_Precio>VD_Costo;
	RETURN Ganancia;
END


-- Procesamieto almacenado utilidad de negocios con codigo ABC
CREATE DEFINER=`root`@`localhost` PROCEDURE `CodigoABC`(
	IN `_FechaDesde` DATE,
	IN `_FechaHasta` DATE
)
LANGUAGE SQL
NOT DETERMINISTIC
READS SQL DATA
SQL SECURITY DEFINER
COMMENT ''
BEGIN
	DECLARE Ganancia DECIMAL(10,2);
	DECLARE GananciaA DECIMAL(10,2);
	DECLARE GananciaB DECIMAL(10,2);
	DECLARE GananciaC DECIMAL(10,2);
	
	OPEN CUR_UTI;

	DECLARE _ProdId INT;
	DECLARE _Utilidad DECIMAL(10,2);
	DECLARE Acumulado DECIMAL(10,2);
	DECLARE Letra CHAR(01);
	DECLARE Final TINYINT DEFAULT 0;
	
	DECLARE CUR_UTI CURSOR FOR
		SELECT VD_PRodId, SUM(VD_Cantidad * (VD_Precio - VD_Costos)) AS Utilidad
			FROM Ventas_Detalle
			JOIN ventas ON VD_VentasID = Ventas_ID
		WHERE Ventas_Fecha BETWEEN _FechaDesde AND _FechaHasta AND 
		VD_Costos >0 AND VD_Precio>VD_Costo
		GROUP BY VD_ProdId
		ORDER BY Utilidad;
		
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET Final=1;
	
	SELECT UtilidadTotal(_FechaDesde, _FechaHasta) INTO Ganancia;
	
	SET GananciaA = Ganancia * .80;
	SET GananciaB = Ganancia * .15;
	SET GananciaC = Ganancia * .05;
	SET Acumulado = 0;
	SET Letra = "C";
	
	WHILE Final=0 DO
		FETCH CUR_UTI INTO _ProdId,_Utilidad;
		IF Final = 0 THEN
			SET Acumulado = Acumulado + _Utilidad;
			IF Letra = "C" AND Acumulado > GananciaC THEN
				SET Acumulado = _Utilidad;
				SET Letra = "B";
			END IF;
			IF Letra = "B" AND Acumulado > GananciaB THEN
				SET Acumulado = _Utilidad;
				SET Letra = "A";
			END IF;
			UPDATE productos SET Prod_ABC = Letra WHERE Prod_Id = _ProdId;
		END IF;
	END WHILE;
	
	CLOSE CUR_UTI;
	
END

-- Trigger para pasar a mayuscula todos los nombres que insertemos en la tabla alumnos
BEGIN
	SET NEW.Alumno_Nombre = UCASE(NEW.Alumno_Nombre);
END

-- Triger para guardar en una tabla los update que hagamos de en este caso el precio, con los paramentros de nuevo ID, viejo precio, nuevo precio, usuario

BEGIN
	IF new.Prod_Precio <> OLD.Prod_Precio THEN
		INSERT INTO productos_historial (PH_ProdId, PH_PrecioANT, PH_PrecioNEW,
						PH_Usuario) VALUES
			(NEW.Prod_Id, OLD.Prod_Precio, NEW.Prod_Precio, CURRENT_USER);
	END IF;
END



--Creación de una vista
ALTER ALGORITHM = UNDEFINED DEFINER=`root`@`localhost` SQL SECURITY DEFINER VIEW `listadeproductos` AS select `productos`.`Prod_Id` AS `Prod_Id`,`productos`.`Prod_Descripcion` AS `Prod_Descripcion`,`productos`.`Prod_Color` AS `Prod_Color`,`productos`.`Prod_Status` AS `Prod_Status`,`productos`.`Prod_Precio` AS `Prod_Precio`,`productos`.`Prod_ProvId` AS `Prod_ProvId`,`productos`.`Prod_ABC` AS `Prod_ABC` from `productos` ;


--MODELO ER
--Creación de base de datos através de MySQL con el modelo entidad-relación
-- MySQL Workbench Forward Engineering

SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0;
SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0;
SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';

-- -----------------------------------------------------
-- Schema VentasSistema
-- -----------------------------------------------------

-- -----------------------------------------------------
-- Schema VentasSistema
-- -----------------------------------------------------
CREATE SCHEMA IF NOT EXISTS `VentasSistema` DEFAULT CHARACTER SET utf8 ;
USE `VentasSistema` ;

-- -----------------------------------------------------
-- Table `VentasSistema`.`Proveedores`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Proveedores` (
  `Prov_Id` INT NOT NULL,
  `Prov_Nombre` VARCHAR(45) NULL,
  `Productos_Prod_Id` INT UNSIGNED NOT NULL,
  `Productos_Proveedores_Prov_Id` INT NOT NULL,
  PRIMARY KEY (`Prov_Id`, `Productos_Prod_Id`, `Productos_Proveedores_Prov_Id`),
  INDEX `fk_Proveedores_Productos1_idx` (`Productos_Prod_Id` ASC, `Productos_Proveedores_Prov_Id` ASC) VISIBLE,
  CONSTRAINT `fk_Proveedores_Productos1`
    FOREIGN KEY (`Productos_Prod_Id` , `Productos_Proveedores_Prov_Id`)
    REFERENCES `VentasSistema`.`Productos` (`Prod_Id` , `Proveedores_Prov_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `VentasSistema`.`Productos`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Productos` (
  `Prod_Id` INT UNSIGNED NOT NULL,
  `Prod_Descripcion` VARCHAR(60) NULL,
  `Prod_Color` VARCHAR(45) NULL,
  `Prod_ProvId` INT NULL,
  `Proveedores_Prov_Id` INT NOT NULL,
  PRIMARY KEY (`Prod_Id`, `Proveedores_Prov_Id`),
  INDEX `fk_Productos_Proveedores_idx` (`Proveedores_Prov_Id` ASC) VISIBLE,
  CONSTRAINT `fk_Productos_Proveedores`
    FOREIGN KEY (`Proveedores_Prov_Id`)
    REFERENCES `VentasSistema`.`Proveedores` (`Prov_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `VentasSistema`.`Clientes`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Clientes` (
  `Cli_Id` INT NOT NULL,
  `Cli_Nombre` VARCHAR(45) NULL,
  PRIMARY KEY (`Cli_Id`))
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `VentasSistema`.`Categorias`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Categorias` (
  `Categ_Id` INT NOT NULL,
  `Categ_Nombre` VARCHAR(45) NULL,
  PRIMARY KEY (`Categ_Id`))
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `VentasSistema`.`Categorias_has_Productos`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Categorias_has_Productos` (
  `Categorias_Categ_Id` INT NOT NULL,
  `Productos_Prod_Id` INT UNSIGNED NOT NULL,
  PRIMARY KEY (`Categorias_Categ_Id`, `Productos_Prod_Id`),
  INDEX `fk_Categorias_has_Productos_Productos1_idx` (`Productos_Prod_Id` ASC) VISIBLE,
  INDEX `fk_Categorias_has_Productos_Categorias1_idx` (`Categorias_Categ_Id` ASC) VISIBLE,
  CONSTRAINT `fk_Categorias_has_Productos_Categorias1`
    FOREIGN KEY (`Categorias_Categ_Id`)
    REFERENCES `VentasSistema`.`Categorias` (`Categ_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_Categorias_has_Productos_Productos1`
    FOREIGN KEY (`Productos_Prod_Id`)
    REFERENCES `VentasSistema`.`Productos` (`Prod_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `VentasSistema`.`Ventas_Enc`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Ventas_Enc` (
  `VentasE_Id` INT NOT NULL,
  `VentasE_Fecha` DATE NULL,
  `Ventas_CliId` INT NULL,
  `VentasE_Total` DECIMAL NULL,
  `Clientes_Cli_Id` INT NOT NULL,
  PRIMARY KEY (`VentasE_Id`, `Clientes_Cli_Id`),
  INDEX `fk_Ventas_Enc_Clientes1_idx` (`Clientes_Cli_Id` ASC) VISIBLE,
  CONSTRAINT `fk_Ventas_Enc_Clientes1`
    FOREIGN KEY (`Clientes_Cli_Id`)
    REFERENCES `VentasSistema`.`Clientes` (`Cli_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `VentasSistema`.`Ventas_Detalles`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Ventas_Detalles` (
  `VentasId` INT NOT NULL,
  `Ventas_EnID` INT NULL,
  `VentasID_ProdId` INT NULL,
  `VentasID_Cantidad` MEDIUMINT(5) NULL,
  `Ventas_Enc_VentasE_Id` INT NOT NULL,
  `Ventas_Enc_Clientes_Cli_Id` INT NOT NULL,
  `Productos_Prod_Id` INT UNSIGNED NOT NULL,
  `Productos_Proveedores_Prov_Id` INT NOT NULL,
  PRIMARY KEY (`VentasId`, `Ventas_Enc_VentasE_Id`, `Ventas_Enc_Clientes_Cli_Id`, `Productos_Prod_Id`, `Productos_Proveedores_Prov_Id`),
  INDEX `fk_Ventas_Detalles_Ventas_Enc1_idx` (`Ventas_Enc_VentasE_Id` ASC, `Ventas_Enc_Clientes_Cli_Id` ASC) VISIBLE,
  INDEX `fk_Ventas_Detalles_Productos1_idx` (`Productos_Prod_Id` ASC, `Productos_Proveedores_Prov_Id` ASC) VISIBLE,
  CONSTRAINT `fk_Ventas_Detalles_Ventas_Enc1`
    FOREIGN KEY (`Ventas_Enc_VentasE_Id` , `Ventas_Enc_Clientes_Cli_Id`)
    REFERENCES `VentasSistema`.`Ventas_Enc` (`VentasE_Id` , `Clientes_Cli_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_Ventas_Detalles_Productos1`
    FOREIGN KEY (`Productos_Prod_Id` , `Productos_Proveedores_Prov_Id`)
    REFERENCES `VentasSistema`.`Productos` (`Prod_Id` , `Proveedores_Prov_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `VentasSistema`.`Proveedores`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Proveedores` (
  `Prov_Id` INT NOT NULL,
  `Prov_Nombre` VARCHAR(45) NULL,
  `Productos_Prod_Id` INT UNSIGNED NOT NULL,
  `Productos_Proveedores_Prov_Id` INT NOT NULL,
  PRIMARY KEY (`Prov_Id`, `Productos_Prod_Id`, `Productos_Proveedores_Prov_Id`),
  INDEX `fk_Proveedores_Productos1_idx` (`Productos_Prod_Id` ASC, `Productos_Proveedores_Prov_Id` ASC) VISIBLE,
  CONSTRAINT `fk_Proveedores_Productos1`
    FOREIGN KEY (`Productos_Prod_Id` , `Productos_Proveedores_Prov_Id`)
    REFERENCES `VentasSistema`.`Productos` (`Prod_Id` , `Proveedores_Prov_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `VentasSistema`.`Precios`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `VentasSistema`.`Precios` (
  `Precios_Id` INT NOT NULL,
  `Precios_ProdId` INT NULL,
  `Precio_Valor` DECIMAL NULL,
  `Productos_Prod_Id` INT UNSIGNED NOT NULL,
  `Productos_Proveedores_Prov_Id` INT NOT NULL,
  `Productos_Prod_Id1` INT UNSIGNED NOT NULL,
  `Productos_Proveedores_Prov_Id1` INT NOT NULL,
  PRIMARY KEY (`Precios_Id`, `Productos_Prod_Id`, `Productos_Proveedores_Prov_Id`),
  INDEX `fk_Precios_Productos1_idx` (`Productos_Prod_Id` ASC, `Productos_Proveedores_Prov_Id` ASC) VISIBLE,
  INDEX `fk_Precios_Productos2_idx` (`Productos_Prod_Id1` ASC, `Productos_Proveedores_Prov_Id1` ASC) VISIBLE,
  CONSTRAINT `fk_Precios_Productos1`
    FOREIGN KEY (`Productos_Prod_Id` , `Productos_Proveedores_Prov_Id`)
    REFERENCES `VentasSistema`.`Productos` (`Prod_Id` , `Proveedores_Prov_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_Precios_Productos2`
    FOREIGN KEY (`Productos_Prod_Id1` , `Productos_Proveedores_Prov_Id1`)
    REFERENCES `VentasSistema`.`Productos` (`Prod_Id` , `Proveedores_Prov_Id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


SET SQL_MODE=@OLD_SQL_MODE;
SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS;
SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS;


--Creación de un Dump de nuestra base de datos con las instrucciones de triggers,rutinas,etc(dump es la trasnferencia de la base de datos con todo nuestro cigo, esto lo hacemos desde la terminal)
EJ:D\curso\mysqldum -u root -p -h localhost --triggers --routines curso > Dump02.sql(nombre de el archivo donde almacenamos el db)
--ahora como realizamos el restort o la restauración
EJ:D\curso\mysqldum -u root -p -h localhost < Dump02.sql


-- Practica SQL Profesional Avanzada

-- Deseamos conocer.Con cuantas 'Comedias' trabajamos en la empresa ?

SELECT COUNT(fc.film_id) AS Comedias
FROM film_category fc
JOIN category c 
ON fc.category_id=c.category_id
WHERE c.name = "Comedy";

-- Cual es el actor o actriz que mas películas ha filmado ?

SELECT a.last_name AS actor_que_mas_peliculas_filmo, COUNT(fa.film_id) AS cantidad
FROM actor a
JOIN film_actor fa
ON a.actor_id=fa.actor_id
GROUP BY actor_que_mas_peliculas_filmo
ORDER BY COUNT(*) DESC
LIMIT 10;

-- Que empleado del staff ha alquilado mas películas de la actriz del ejercicio 2 ?

SELECT s.staff_id,s.first_name,s.last_name,COUNT(s.staff_id) AS Cantidad_Vista
FROM staff s
	JOIN rental r ON r.staff_id = s.staff_id
	JOIN inventory i ON r.inventory_id = i.inventory_id
	JOIN film f ON f.film_id = i.film_id
	JOIN film_actor fa ON fa.film_id = f.film_id
WHERE fa.actor_id = 107
GROUP BY s.staff_id;

-- cuantas películas con rating PG-13 que hay en el store nro 2 ha alquilado el mejor cliente de todos ??

SELECT COUNT(*) AS Peliculas
FROM film f
	JOIN inventory i ON i.film_id = f.film_id
	JOIN rental r ON i.inventory_id = r.inventory_id
WHERE f.film_id IN ( SELECT i.film_id
							FROM film f
								JOIN inventory i ON f.film_id = i.film_id
							WHERE f.rating = 'PG-13' AND i.store_id = 2
							GROUP BY i.film_id) AND
		r.customer_id IN (SELECT ren.customer_id FROM (SELECT r.customer_id FROM rental r
								GROUP BY r.customer_id
								ORDER BY COUNT(r.customer_id) DESC
								LIMIT 1) AS ren) AND
		i.store_id=2;	
