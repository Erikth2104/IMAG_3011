Når skyldes avviket at modellen ikke passer målingene, og når tyder kontrollene på problemer i beregningen?
- Vi bruker 3 forskjellige ting for å måle dette $|Q^TQ-I|_f$, $|A-QR|_f$ og residualen $r=b-Ac$.
- Hvis modellen ikke passer målingene får vi høy residual, dette kan skyldes støy i $b$, måleverdiene. 
- Kontrollen kan tyde på problemer i beregningen, hvis en av de to kontrollene er høy. De måler ortonormalitet og rekonsturksjon av selve faktoriseringen $A=QR$
- Når vi økte støyet gikk residual økte også residual normen (del3 oppgave 3).
- Når vi byttet basis fra monomial til chebyshev, fikk vi lavere ortogonalitets feil og lavere maks avvik. Her ser vi at det skyldes feil i beregnignen.

Hva forteller residual, ortogonalitet og kurvefeil hver for seg?
- Residualen ($r=b-Ac)$ sier noe om forskjellen mellom målignene $b$ og modellverdiene i målepunktene.  En liten residual betyr at vi passer godt i målepunktene, hvis den er høy passer vi dårlig i målepunktene. Sier ingenting om det som skjer mellom punktene, så vi kan fortsatt ha stor kurveavvik 
- ortogonalitet ($|Q^TQ-I|_f$) denne sjekker om kolonnene i $Q$ er ortonormale, den sier noe om numerisk pålitetlighet. 
- Kurvefeilen sier noe om avviket fra den sanne kurven fra model kurven langs hele intervallet, 

Hva endret du i redningsforsøket, og hvilke resultater støtter vurderingen?
- I forsøket endre jeg målepunktene, fra jevnt fordelt på intervallet til cosinus fordelt (chebyshev punkter). 
- Vi fikk lavere maks avvik og kondisjonstall, men høyere residual. 
- Dette betyr at modellen passer bedre til den sanne kurven, men vi får høyere feil i målepunktene, når det var støy i de. 

Hvilken begrensning ved forsøket gjør at du bør være forsiktige med å generalisere?
- Vi sjekket ikke veldig mange seeds, så det kan være noe varriasjon fra seed til seed. 
- Vi sjekket også bare et referanse polynom, en grad, og et sett punkter. 