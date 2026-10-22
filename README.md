% 1. Configuración de vectores de Generación (Modelo analítico)
horas = 1:24;

% Generación Solar (Curva sinusoidal de 6:00 a 18:00 horas)
Pmax_solar = 25;
solar = zeros(1, 24);
h_dia = 6:18;
solar(h_dia) = Pmax_solar .* sin((pi/12) .* (h_dia - 6));

% Generación Eólica (Valores aleatorios controlados)
Pmax_eolica = 15;
rng(1); % Semilla para reproducibilidad
eolica = Pmax_eolica .* rand(1, 24);

% 2. Demandas individuales fijas por cada carga (kW)
escuela_base = [1 1 1 1 1 2 2 2 2 3 3 2 2 2 3 3 3 3 3 1 1 1 1 1];
bombeo_base  = [1 1 1 0 1 2 2 3 3 3 3 3 2 3 3 4 4 4 3 1 1 1 1 1];
alumbrado_p  = [1 0.5 1.5 1 1 2 3 3 4 4 3 3 3 4 4 5 8 8 8 2 1 0.5 1 1.5];

capacidad_bateria = 100; % Capacidad total en kWh[cite: 4]
SOC = 50;                % Estado de carga inicial (50%)[cite: 4]

gen_total_vec = zeros(1, 24);
demanda_real_vec = zeros(1, 24);

fprintf('REPORTE DETALLADO DE CONSUMO Y CARGAS - MICRORRED (24 HORAS)\n');
fprintf('--------------------------------------------------------------------------------------------------------------------\n');
fprintf('Hora | Solar | Eolica | Gen.Tot | Escuela | Bombeo | Alumbrado | Dem.Total | Balance |  SOC  | Estado / Accion\n');
fprintf('--------------------------------------------------------------------------------------------------------------------\n');

for t = 1:24
    gen_tot = solar(t) + eolica(t);
    gen_total_vec(t) = gen_tot;

    % Asignación de demandas base para la hora actual
    p_escuela = escuela_base(t);
    p_bombeo  = bombeo_base(t);
    p_alumbrado = alumbrado_p(t);

    % 3. Lógica de priorización y deslastre de cargas según los rangos del SOC
    if SOC > 40
        estado = 'Ruta Segura';
        demanda_actual = p_escuela + p_bombeo + p_alumbrado;

    elseif SOC > 20 && SOC <= 40
        estado = 'Ruta Alerta';
        p_alumbrado = 0; % Se desconecta el alumbrado público (baja prioridad)[cite: 1]
        demanda_actual = p_escuela + p_bombeo + p_alumbrado;

    else
        estado = 'Ruta Emergencia';
        p_bombeo = 0;    % Se desconecta el bombeo y el alumbrado[cite: 1]
        p_alumbrado = 0; 
        demanda_actual = p_escuela + p_bombeo + p_alumbrado; % Conserva únicamente la escuela[cite: 1]
    end

    demanda_real_vec(t) = demanda_actual;

    % 4. Balance energético neto
    balance = gen_tot - demanda_actual;

    % 5. Actualización del Estado de Carga (SOC) de la batería
    if balance >= 0
        excedente = balance;
        SOC = SOC + (excedente / capacidad_bateria) * 100;
        if SOC > 100, SOC = 100; end
    else
        faltante = abs(balance);
        SOC = SOC - (faltante / capacidad_bateria) * 100;
        if SOC < 0, SOC = 0; end
    end

    % 6. Reporte detallado por consola en una sola ejecución limpia
    fprintf('%4d | %5.1f | %6.1f | %7.1f | %7.1f | %6.1f | %9.1f | %9.1f | %7.1f | %5.1f%% | %s\n', ...
        t, solar(t), eolica(t), gen_tot, p_escuela, p_bombeo, p_alumbrado, demanda_actual, balance, SOC, estado);
end

% 7. Generación de la gráfica de rendimiento
figure;
plot(horas, gen_total_vec, '-o', 'LineWidth', 2, 'Color', '#77AC30', 'DisplayName', 'Generación Total (Solar + Eólica)');
hold on;
plot(horas, demanda_real_vec, '-s', 'LineWidth', 2, 'Color', '#DB4D4D', 'DisplayName', 'Demanda Real de la Comunidad');
grid on;
xlabel('Tiempo (Horas)', 'FontSize', 11, 'FontWeight', 'bold');
ylabel('Potencia (kW)', 'FontSize', 11, 'FontWeight', 'bold');
title('Perfil Energético Diario de la Microrred con Deslastre de Cargas', 'FontSize', 12, 'FontWeight', 'bold');
legend('Location', 'northeast');
xlim([1 24]);
hold off;
