%% 555 Timer Clock Analysis for a Drag Strip Timer
% Calculates the ideal 555 astable clock, then uses a Monte Carlo
% simulation to show how component tolerances change the displayed race time.

clear; clc; close all;
rng(1);                          % fixed seed so results are repeatable

%% 1. Inputs (replace with your own component values)
R1 = 1e3;                        % ohms
R2 = 6.8e3;                      % ohms
C  = 10e-6;                      % farads
tolR = 0.05;                     % resistor tolerance (+/- 5%)
tolC = 0.10;                     % capacitor tolerance (+/- 10%)
targetF  = 10;                   % desired clock in Hz (one tick = 0.1 s)
raceTime = 5.000;                % true race time in seconds
N = 10000;                       % number of simulated circuits

%% 2. Ideal 555 astable calculations
tHigh = 0.693 * (R1 + R2) * C;   % output-high time (s)
tLow  = 0.693 * R2 * C;          % output-low time (s)
T     = tHigh + tLow;            % full period (s)
f     = 1 / T;                   % frequency (Hz)
duty  = tHigh / T * 100;         % duty cycle (%)

fprintf('Ideal clock\n');
fprintf('  Period:     %.4f s\n', T);
fprintf('  Frequency:  %.3f Hz (target %.1f Hz)\n', f, targetF);
fprintf('  Duty cycle: %.1f %%\n\n', duty);

%% 3. Monte Carlo: random component values within tolerance
R1s = R1 * (1 + tolR * (2 * rand(N, 1) - 1));
R2s = R2 * (1 + tolR * (2 * rand(N, 1) - 1));
Cs  = C  * (1 + tolC * (2 * rand(N, 1) - 1));

fs = 1 ./ (0.693 * (R1s + 2 * R2s) .* Cs);   % frequency of each circuit

%% 4. What the display would show for a 5.000 s race
% The counter counts clock ticks, and the display assumes each tick is
% exactly 1/targetF seconds, so a fast or slow clock gives a wrong time.
ticks     = floor(raceTime * fs);            % ticks counted in the race
displayed = ticks / targetF;                 % time the display reports
errorSec  = displayed - raceTime;            % display error in seconds

fprintf('Monte Carlo (%d circuits)\n', N);
fprintf('  Clock frequency: mean %.2f Hz, min %.2f Hz, max %.2f Hz\n', ...
        mean(fs), min(fs), max(fs));
[~, worst] = max(abs(errorSec));             % index of the largest error
fprintf('  Display error:   mean %+.3f s, worst case %+.3f s\n', ...
        mean(errorSec), errorSec(worst));
fprintf('  Within +/- 0.1 s: %.1f %% of circuits\n', ...
        100 * mean(abs(errorSec) <= 0.1));

%% 5. Plots
figure('Name', '555 Clock Analysis', 'Position', [100 100 900 400]);

subplot(1, 2, 1);
histogram(fs, 50);
xline(targetF, 'r--', 'Target', 'LineWidth', 1.5);
xlabel('Clock frequency (Hz)');
ylabel('Number of circuits');
title('Clock frequency spread');
grid on;

subplot(1, 2, 2);
histogram(displayed, 50);
xline(raceTime, 'r--', 'True time', 'LineWidth', 1.5);
xlabel('Displayed time (s)');
ylabel('Number of circuits');
title(sprintf('Displayed time for a %.3f s race', raceTime));
grid on;
