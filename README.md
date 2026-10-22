# CODIGO-MATLAB
UNAD RETO · E5
% =========================================================================
% SIMULACIÓN DEL CONTROLADOR INTELIGENTE DE MICRO-RED ELÉCTRICA (SIGE)
% =========================================================================

clear; clc;

% 1. CONFIGURACIÓN INICIAL Y PARÁMETROS
Capacidad_bat = 100; % Capacidad total en kWh
SOC = 50.0;          % Estado de carga inicial (%)

% Vectores de 24 horas (Reemplaza con tus datos del Reto 2 si son diferentes)
E_solar    = [0, 0, 0, 0, 0, 5, 12, 20, 28, 35, 40, 42, 40, 32, 25, 15, 5, 0, 0, 0, 0, 0, 0, 0];
E_eolica   = [10, 12, 11, 10, 8, 7, 6, 8, 10, 12, 10, 8, 9, 11, 12, 14, 15, 16, 18, 15, 12, 10, 8, 9];
E_demanda  = [15, 14, 12, 12, 15, 20, 25, 30, 32, 35, 38, 40, 38, 35, 30, 28, 32, 40, 45, 42, 35, 28, 20, 18];

% Encabezado de la consola
fprintf('====================================================================================\n');
fprintf(' Hora |  Gen (kWh) |  Dem (kWh) | Balance  |  SOC (%%) | Estado Escuela | Estado Bombeo | Estado Alumbrado\n');
fprintf('====================================================================================\n');

% 2. BUCLE DE SIMULACIÓN DE 24 HORAS
for hora = 1:24
    E_gen = E_solar(hora) + E_eolica(hora);
    E_dem = E_demanda(hora);
    
    % Evaluar balance
    if E_gen >= E_dem
        % Superávit de energía
        excedente = E_gen - E_dem;
        SOC = SOC + (excedente / Capacidad_bat) * 100;
        
        if SOC > 100
            SOC = 100;
        end
        
        escuela = 'ACTIVA'; bombeo = 'ACTIVO'; alumbrado = 'ACTIVO';
        tipo_balance = 'SUPERÁVIT';
        
    else
        % Déficit de energía
        faltante = E_dem - E_gen;
        SOC = SOC - (faltante / Capacidad_bat) * 100;
        
        if SOC < 0
            SOC = 0;
        end
        
        tipo_balance = 'DÉFICIT  ';
        
        % Algoritmo de Load Shedding según SOC
        if SOC > 40
            % Ruta Segura
            escuela = 'ACTIVA'; bombeo = 'ACTIVO'; alumbrado = 'ACTIVO';
        elseif SOC > 20 && SOC <= 40
            % Ruta de Alerta
            escuela = 'ACTIVA'; bombeo = 'ACTIVO'; alumbrado = 'SUSPENDIDO';
        else
            % Ruta de Emergencia (SOC <= 20)
            escuela = 'ACTIVA'; bombeo = 'SUSPENDIDO'; alumbrado = 'SUSPENDIDO';
        end
    end
    
    % Impresión de reporte por hora
    fprintf('  %02d  |   %6.2f   |   %6.2f   | %s |  %6.2f%% |     %s    |    %s   |    %s\n', ...
            hora, E_gen, E_dem, tipo_balance, SOC, escuela, bombeo, alumbrado);
end
fprintf('====================================================================================\n');
