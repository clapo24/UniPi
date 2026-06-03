# Comani Unix
| COMANDO | DESCRIZIONE |
| --- | --- |
| setxkbmap it | tastiera in italiano |
|cd | cambio directory (cartella) |
| cd .. | esco dalla cartella |
| -- | -- |
| touch prova| crea file vuoto |
| touch prova.cpp | crea file vuoto.cpp |
| ls | elenca i contenuti della cartella |
| clear | cancella il terminal |
| -- | -- |
| g++ main.cpp -o main | crea/salva eseguibile |
| ./main | esegue |
| ./main < input0| esegue il main passando input0 |
| ./main < input0 \| diff - output0 | verifica se input e output sono coincidenti |
| -- | -- |
| python3 tester.py ./main | eseguire il file python per TestSet |

# Include standard

``` cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;
``` 

# Alberi binari di ricerca
non bilanciato --> inserimento ricorsivo
``` cpp
struct Nodo { //2014-05-29 forse
    int chiave;
    string valore;
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int k, const std::string& v)
        : chiave(k), valore(v), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
private:
    Nodo* radice;

    // Questa funzione restituisce il nodo radice del sottoalbero modificato.
    Nodo* insertRecursive(Nodo* current, int k, const string& v) {
        
        if (current == nullptr) // caso base
            return new Nodo(k, v);

        if (k < current->chiave) 
            current->sinistro = insertRecursive(current->sinistro, k, v);
        else if (k > current->chiave) 
            current->destro = insertRecursive(current->destro, k, v);
        else 
            //gestione dei duplicati

        return current;
    }

public:
    BinTree() : radice(nullptr) {}
    Nodo* getRoot() { return radice;}

    // Funzione pubblica per avviare l'inserimento ricorsivo
    void insert(int k, std::string v) {
        radice = insertRecursive(radice, k, v);
    }
    
};
```

versione dell'albero binario non bilanciato --> con inserimento iterativo
``` cpp 
struct Nodo { // 2014-06-10
    int chiave;
    string valore;
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int k, const string& v) : chiave(k), valore(v), sinistro(nullptr), destro(nullptr) {}
};

class BinTree{
	Nodo* radice;
public:
	BinTree(){ radice = nullptr; }
	Nodo* getRoot(){ return radice; }

	void inserisci(int k, std::string v) {
		if (radice == nullptr) { // albero vuoto
		    radice = new Nodo(k, v);
		    return;
		}
		
		Nodo* current = radice;
		Nodo* parent = nullptr; // Tiene traccia del nodo genitore

		while (current != nullptr) {
		    parent = current; // Aggiorna il genitore al nodo corrente

		    if (k <= current->chiave)
		        current = current->sinistro; // Vai a sinistra
		    else if (k > current->chiave)
		        current = current->destro; // Vai a destra
		}
		
		if (k <= parent->chiave) {
		    parent->sinistro = new Nodo(k, v);
		} else { // k > parent->chiave
		    parent->destro = new Nodo(k, v);
		}
    	}
    	
    	void trovaNodiAProfondita(Nodo* nodo, int profondita_corrente, int D, vector<string>& risultato) {
		if (nodo == nullptr) return;
		
		if (profondita_corrente == D) {
		    risultato.push_back(nodo->valore); //push_back è funzine della libreria vector --> aggiunge elemento alla fine del vector con complessità O(1)
		    return; // Non serve continuare più in profondità
		}
		
		// Continua la visita sui figli
		trovaNodiAProfondita(nodo->sinistro, profondita_corrente + 1, D, risultato);
		trovaNodiAProfondita(nodo->destro, profondita_corrente + 1, D, risultato);
    }
    
	// Funzione pubblica per avviare la ricerca
	vector<string> cercaStringheAProfondita(int D) {
        	vector<string> risultato;
        	trovaNodiAProfondita(radice, 0, D, risultato);
        	return risultato;
    	}
};


int main(){
	// inserimento di N
	int N;
	cin >> N;
	
	// inserimento di D
	int D;
	cin >> D;
	
	// creo albero
	BinTree albero;
	
	// sequenza di N coppie
	for(int i = 0; i<N; ++i){
		int intero;
		string stringa;
		cin >> intero;
		cin >> stringa;
		albero.inserisci(intero, stringa);
	}
	
	// trova tutte le stringhe a profonsità D
	vector<string> stringhe;
	stringhe = albero.cercaStringheAProfondita(D);
	
	// Ordina le stringhe lessicograficamente
	sort(stringhe.begin(), stringhe.end()); //sort --> funzione di algorithm
    
	// Output secondo le specifiche
	if (stringhe.empty()) //stringhe.empty() //stringa vuota
		cout << 0 << endl;
	else {
		cout << stringhe.size() << endl;
		for (int i = 0; i < stringhe.size(); ++i) 
    			cout << stringhe[i] << endl;
	}
			
	return 0;
	
}
```

altro esame su albero binario di ricerca non bilanciato 
``` cpp
struct Nodo { //2014-07-02
    int valore;
    
    int I;                // somma delle chiavi dei nodi interni del sottoalbero
    int F;                // somma delle chiavi delle foglie del sottoalbero
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v) : valore(v), I(0), F(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;

    /* ---------- inserimento standard in un BST ---------- */
    void inserisciNodo(Nodo*& u, int v) {
        if (u == nullptr) {
            u = new Nodo(v);
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v);
        else                 
        	inserisciNodo(u->destro,  v);
    }

    /* ---------- passata post‑order: calcola I(u) e F(u) ---------- */
    //visita postOrder --> sinistra, destra, nodo
    void calcolaIF(Nodo* u, int& I_out, int& F_out) {
        if (u == nullptr) {
            I_out = F_out = 0;
            return;
        }

        int Isx = 0, Fsx = 0;
        int Idx = 0, Fdx = 0;

        calcolaIF(u->sinistro, Isx, Fsx);
        calcolaIF(u->destro,  Idx, Fdx);

        bool foglia = (u->sinistro == nullptr && u->destro == nullptr);

        if (foglia) { //se è foglia
            u->I = 0;
            u->F = u->valore;
        } else { //se non è foglia
            u->I = u->valore + Isx + Idx;
            u->F = Fsx + Fdx;
        }

        I_out = u->I;
        F_out = u->F;
    }
    // alla fine di questa funzine tutti i nodi hanno il proprio valore di I e F

    /* ---------- passata in‑order: stampa le chiavi ammissibili ---------- */
    // la visita inOrder permette di stampare in senso crescente --> sinistro, nodo, destro
    void stampaValidi(Nodo* u) const {
        if (u == nullptr) 
        	return;
        	
        stampaValidi(u->sinistro);
        
        if (u->I <= u->F) 
        	cout << u->valore << ' ';
        	
        stampaValidi(u->destro);
    }

public:
    BinTree() : radice(nullptr) {}

    void inserisci(int v) { inserisciNodo(radice, v); }

    void risolvi_e_stampa() {
        int Iroot = 0, Froot = 0;
        calcolaIF(radice, Iroot, Froot);
        stampaValidi(radice);
        cout << '\n';
    }
};

int main() {
	// inserimento di N
	int N;
	cin >> N;
	
	//creazione albero
	BinTree albero;
	
	//inserimento dei noi
	for (int i=0; i<N; ++i){
		int valore;
		cin >> valore;
		albero.inserisci(valore);
	}
	
	albero.risolvi_e_stampa();
	
	return 0;
}

```

albero binario di ricerca non bilanciato 
``` cpp
struct Nodo { // 2015-01-12
 	int valore;
 	Nodo* sinistro;
 	Nodo* destro;
 	
 	Nodo(int x): valore(x), sinistro(nullptr), destro(nullptr) {}
 };
 
 class BinTree{
 	Nodo* radice;
 	
 	void inserisciNodo(Nodo*& u, int v) {
        if (u == nullptr) { //caso base
            u = new Nodo(v);
            return;
        }
        
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v);
        else                 
        	inserisciNodo(u->destro,  v);
    }
    
    bool isOK(Nodo * tree, int & maxH) {
    	if(tree == 0){
    		maxH=0;
    		return true;
    	}
    	
    	int hSX = 0;
    	int hDX =0;
    	
    	bool propSX = isOK(tree->sinistro, hSX);
    	bool propDX = isOK(tree->destro, hDX);
    	
    	maxH = 1 + max(hSX, hDX); //max(a,b) --> funzione di algorithm, ritorna il massimo tra a e b
    	
    	if(abs(hSX-hDX) > 1) //abs(x) --> funzione di bho, ritorna il valore assoluto di x
    		return false;
    	if(propSX == true && propDX==true)
    		return true;
    	
    	return false;
    }
    
public:
	BinTree(){ radice = nullptr; }
	
	void inserisci(int v) { inserisciNodo(radice, v); }
	
	bool proprieta() {
		int altezza=0;	
		return isOK(radice, altezza);
	}
	
 };
 
 
 int main() {
 	//inserimento di N
 	int N;
 	cin >> N;
 	
 	//creazione albero
 	BinTree albero;
 	
 	// inserimento dei nodi
 	for (int i=0; i<N;  ++i){
 	int x;
 	cin >> x;
 	albero.inserisci(x);
 	}
 	
 	bool variabile = albero.proprieta();
 	
 	if (variabile==true)
 		cout << "ok" << endl;
 	else
 		cout << "no" << endl;
 	
 	return 0;
 }
```


``` cpp
struct Nodo { //2015-01-28
    int valore;
    
    int D;                // somma delle etichette el sottoalbero destro
    int S;                // somma delle etichette el sottoalbero sinistro
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v) : valore(v), D(0), S(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;

    /* ---------- inserimento standard in un BST ---------- */
    void inserisciNodo(Nodo*& u, int v) { //complessità O(h)
        if (u == nullptr) { // caso base --> ho trovato buco per inserimento
            u = new Nodo(v);
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v);
        else                 
        	inserisciNodo(u->destro,  v);
    }

    /* ---------- passata post‑order: calcola S(u) e D(u) ---------- */
    //visita postOrder --> sinistra, destra, nodo
    int calcolaSD(Nodo* u, int k) {
        if (u == nullptr) //caso base --> foglia
            return 0;

        int Sx = calcolaSD(u->sinistro, k); // S(u) //visita sottoalbero sinistro
        int Dx = calcolaSD(u->destro, k); // D(u) //visita sottoalbero destro
        
        u->S = Sx; // ora ho le somme dei figli
        u->D = Dx;
	
	return Sx + Dx + u->valore; //restituisco la somma totale
    }
    // alla fine di questa funzine tutti i nodi hanno il proprio valore di S e D

    /* ---------- passata in‑order: stampa le chiavi ammissibili ---------- */
    // la visita inOrder permette di stampare in senso crescente --> sinistro, nodo, destro
    void stampaValidi(Nodo* u, int k) const {
        if (u == nullptr) 
        	return;
        	
        stampaValidi(u->sinistro, k);
        
        if ((u->S * k) < u->D) 
        	cout << u->valore << endl;
        	
        stampaValidi(u->destro, k);
    }

public:
    BinTree() : radice(nullptr) {}

    void inserisci(int v) { inserisciNodo(radice, v); }

    void risolvi_e_stampa(int k) { //complessità O(n) complessiva --> ogni nodo è toccato esattamente due volte
        calcolaSD(radice, k);
        stampaValidi(radice, k);
    }
};

int main() {
	// inserimento di N
	int N;
	cin >> N;
	
	//inserimento di k
	int k;
	cin >> k; 
	
	//creazione albero
	BinTree albero;
	
	//inserimento dei noi
	for (int i=0; i<N; ++i){
		int valore;
		cin >> valore;
		albero.inserisci(valore);
	}
	
	albero.risolvi_e_stampa(k);
	
	return 0;
}

```


``` cpp
struct Nodo { // 2015-07-20
    int valore;
    int altezza_nodo;
                 
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v, int h) : valore(v), altezza_nodo(h), sinistro(nullptr), destro(nullptr) {}
};

struct Coppia {
    int conteggio; //conteggio corrisponde al numero di nodi ad altezza h per il rispettivo albero ID
    int id;
};

class BinTree {
    Nodo* radice;
    int altezza_albero;

    /* ---------- inserimento standard in un BST ---------- */
    void inserisciNodo(Nodo*& u, int v, int h) { //complessità O(h)
        if (u == nullptr) { // caso base --> ho trovato buco per inserimento
            u = new Nodo(v, h);
            if (h > altezza_albero)
            	altezza_albero=h;
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, h+1);
        else                 
        	inserisciNodo(u->destro, v, h+1);
        
    }
    
    int contaNodiPrivato(Nodo* u, int h_targhet, int h_current){
    		if(u==nullptr || h_current > h_targhet)
    			return 0; // caso base 1
    		if(h_current == h_targhet) 
    			return 1; //caso base 2 --> altezza corretta
    		
    		return contaNodiPrivato(u->sinistro, h_targhet, h_current+1) + contaNodiPrivato(u->destro, h_targhet, h_current+1); //ricorsione
    }

    

public:
    BinTree(){ 
    	radice = nullptr; 
    	altezza_albero = -1;
    	}

    void inserisci(int v) { 
    	inserisciNodo(radice, v, 0); 
    	}
    
    int getAltezza() { return altezza_albero; }
    
    int conta_nodi_ad_altezza_h(int h) {
    	return contaNodiPrivato(radice, h, 0);
    }
    
};

int main() {
	// inserimento di N e D
	int N;
	int D;
	cin >> N >> D;
	
	
	//creazione vettore lungo D di alberi 
	vector<BinTree> albero(D); //D alberi, indici 0 ... D-1
	
	//inserimento dei noi nei D alberi
	for (int i=0; i<N; ++i){
		int valore;
		int id;
		cin >> valore >> id;
		albero[id].inserisci(valore); //seleziono albero corretto e inserisco valore
	}
	
	int h_min;
	//scorriamo gli D alberi e troviamo altezza minima
		//altezza mimina primo albero
	h_min=albero[0].getAltezza();
	int altezza;
	for(int id=1; id<D; ++id){
		altezza=albero[id].getAltezza();
		if(altezza<h_min)
			h_min=altezza;
	} //interno del for poteva essere scritto come h_min = min(h_min, albero[id].getAltezza());
	
	
	//crea array di coppie (conteggio, ID)
	vector<Coppia> conteggi_e_ID;
	for(int id = 0; id < D; ++id) {
		Coppia c;
		c.conteggio = albero[id].conta_nodi_ad_altezza_h(h_min);
		c.id = id;
		conteggi_e_ID.push_back(c); //inserisce 'c' in fondo al vector
	}
	
	// Ordina con sort(inizio, fine, comparatore)
	sort(conteggi_e_ID.begin(), conteggi_e_ID.end(), 
				[](const Coppia& a, const Coppia& b) {
				if (a.conteggio != b.conteggio)
					return a.conteggio > b.conteggio;  // decresecnte per conteggio
				return a.id > b.id;                    // parità: decrescente per ID
	});
	    
	// Stampa risultati
	for (size_t i = 0; i < conteggi_e_ID.size(); ++i) {
		cout << conteggi_e_ID[i].id << endl;
	}
	

	
	return 0;
}
```

``` cpp
struct Nodo { // 2016-01-11
    int valore;
                 
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v) : valore(v), sinistro(nullptr), destro(nullptr) {}
};

struct albero_e_altezza {
	int ID;
	int altezza;
};

class BinTree {
	Nodo* radice;
	int altezza_albero;

	/* ---------- inserimento standard in un BST ---------- */
	void inserisciNodo(Nodo*& u, int v, int h) { //complessità O(h)
		if (u == nullptr) { // caso base --> ho trovato buco per inserimento
			u = new Nodo(v);
			if (h > altezza_albero)
				altezza_albero=h;
			return;
		}
		
		if (v <= u->valore) 
			inserisciNodo(u->sinistro, v, h+1);
		else                 
			inserisciNodo(u->destro, v, h+1);      
	}
	
	void inOrder_merge(Nodo*& u, BinTree& albero_high){ //u sono i nodi di albero_low //
		if (u == nullptr) 
        		return;
        	
        	inOrder_merge(u->sinistro, albero_high);
        
        	// devo inserire questo nodo nell'albero high
        	albero_high.inserisci(u-> valore); 
        	
        	inOrder_merge(u->destro,albero_high);
	}
	
	void inOrder_stampa_foglie(Nodo* & u){ //u sono i nodi di albero_high
		if(u == nullptr)
			return;
		
		inOrder_stampa_foglie(u->sinistro);
		
		if(u->sinistro==nullptr && u->destro==nullptr)
			cout << u->valore << endl;
		
		inOrder_stampa_foglie(u->destro);
	}
	
public:
	BinTree(){ 
    		radice = nullptr; 
    		altezza_albero = -1;
    	}

    	void inserisci(int v) { 
    		inserisciNodo(radice, v, 0); 
    	}
    
    	int getAltezza() { return altezza_albero; }
    	
    	void trasferisci(BinTree& albero_hight){
    		inOrder_merge(radice, albero_hight); //radice sarebbe this->radice, quindi è la radice di albero_low
    		albero_hight.inOrder_stampa_foglie(albero_hight.radice);
    	}

};


int main() {
	// inserimento di N e D
	int N;
	int D;
	cin >> N >> D;
	
	
	//creazione vettore lungo D di alberi 
	vector<BinTree> albero(D); //D alberi, indici 0 ... D-1
	
	//inserimento dei noi nei D alberi
	for (int i=0; i<N; ++i){
		int valore;
		int id;
		cin >> valore >> id;
		albero[id].inserisci(valore); //seleziono albero corretto e inserisco valore
	}
	
	// vettore con alberi in ordine corretto per ottenre low e hight
	vector<albero_e_altezza> coppia(D);
		//riempimento vector
	for(int i=0; i<D; ++i) //i coincide con id, tutto ok
		coppia[i] = {i, albero[i].getAltezza()};
		//ordinamento
	sort(coppia.begin(), coppia.end(), 
			[](const albero_e_altezza& a, const albero_e_altezza& b){
			if(a.altezza != b.altezza)
				return a.altezza < b.altezza; //crescente
			// stessa altezza
			return a.ID < b.ID; //crescente per ID
			});
	// probailmente snon serviva completare l'ordinamento --> potevo trovare low e high con complessità minore
	
	int low = coppia[0].ID;
	int high = coppia[D-1].ID;
	
	//creazione albero risultante --> trasferisco nodi da low a high e stampo foglie risultanti
	albero[low].trasferisci(albero[high]);
	
	return 0;
}
```

``` cpp
struct Nodo { //2016-06-08
    int ID; //etichetta di riferimetno per inserimento 
    int peso; // P
    int Cmax; // carico massimo
    int carico; //carico di un nodo è definito come la somma dei pesi di tutti i nodi del suo sottoalbero
                 
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int id, int p, int Cmax) : ID(id), peso(p), Cmax(Cmax), carico(0), sinistro(nullptr), destro(nullptr) {}
};


class BinTree {
    Nodo* radice;

    /* ---------- inserimento standard in un BST ---------- */
    void inserisciNodo(Nodo*& u, int id, int p, int Cmax) { //complessità O(h)
        if (u == nullptr) { // caso base --> ho trovato buco per inserimento
            u = new Nodo(id, p, Cmax);
            return;
        }
        if (id <= u->ID) 
        	inserisciNodo(u->sinistro, id, p, Cmax);
        else                 
        	inserisciNodo(u->destro, id, p, Cmax);
    }
    
    int calcoloCarico_Verifica(Nodo* u, vector<int>& violati){ //faccio una visita postOrder --> guardo sinistro, destro e poi me
    			// non passo nodo per riferimento perchè noi vogliamo modificare i valori del nodo puntato, non a cosa punta il puntatore
    	if (u == nullptr)
    		return 0;
    	
    	int pesoSinistro = calcoloCarico_Verifica(u->sinistro, violati);
    	int pesoDestro = calcoloCarico_Verifica(u->destro, violati);
    	
    	u->carico = pesoSinistro + pesoDestro;
    	
    	if(u->carico > u->Cmax)
    		violati.push_back(u->ID);
    	
    	return u->peso + u->carico;
    	
    }
   
    

public:
    BinTree(){ 
    	radice = nullptr;
    	}

    void inserisci(int id, int peso, int Cmax) { 
    	inserisciNodo(radice, id, peso, Cmax); 
    	}
    
    void gestioneCarico(vector<int>& violati){
    	calcoloCarico_Verifica(radice, violati);
    }
    
    
};

int main() {
	// inserimento di N
	int N;
	cin >> N;
	
	
	//creazionedell'albero
	BinTree albero;
	
	//inserimento dei nodi nell' albero
	for (int i=0; i<N; ++i){
		int id;
		int peso;
		int Cmax;
		cin >> id >> peso >> Cmax;
		albero.inserisci(id, peso, Cmax); 
	}
	
	vector<int> violati;
	albero.gestioneCarico(violati);
	
	if(violati.empty()) //corrisponde a --> vector.size() == 0
		cout << "ok" << endl;
	else {
		cout << "no" << endl;
		sort(violati.begin(), violati.end()); //ordine crescente
		//stampo
		for(int i=0; i < violati.size(); ++i)
			cout << violati[i] << endl;	
	}

	
	return 0;
}
```

``` cpp
// 2016-06-29
``` 

``` cpp
struct Nodo { // 2016-07-20
	int chiave;
	bool mediano;
	
	int altezza_nodo; //corrisponde alla distanza tra il nodo e la radice
	int lx; //distanza massima dalla foglia del sottoalbero
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int c, int h) : chiave(c), altezza_nodo(h), mediano(false), lx(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree{
	Nodo* radice;
	
	void inserisciNodo(Nodo*& u, int chiave, int h){
		if (u == nullptr){
			u = new Nodo(chiave, h);
			return;
		}
		
		if (chiave <= u->chiave)
			inserisciNodo(u->sinistro, chiave, h+1);
		else
			inserisciNodo(u->destro, chiave, h+1);
	}
	
	int calcola_lx(Nodo* u){ //come se facessi una visita postOrder --> complessità O(n)
		if (u == nullptr)
			return -1;
		
		int sx = calcola_lx(u->sinistro);
		int dx = calcola_lx(u->destro);
		
		u->lx = 1 + max(sx, dx);
		
		if (abs(u->altezza_nodo - u->lx) <= 1)
			u->mediano = true;
		
		return u->lx;
	}
	
	void stampa_mediani(Nodo* u, int& stampati, int k){
		if (u == nullptr || stampati >= k) //finiti i mediani o raggiunto k
			return;
			
		// visita inOrder per stampare mediani in ordine crescente
		stampa_mediani(u->sinistro, stampati, k);
		
		if (stampati < k && u->mediano == true){
			cout << u->chiave << endl;
			stampati++;
		}
		
		stampa_mediani(u->destro, stampati, k);
	}
public: 
	BinTree(){
		radice=nullptr;
	}
	
	void inserisci(int chiave){
		inserisciNodo(radice, chiave, 0);
	}
	
	void calcola_mediani(){
		calcola_lx(radice);
	}
	
	void stampa_primi_k_mediani(int k){
		int stampati=0;
		stampa_mediani(radice, stampati, k);
	}
};

int main(){
	//inserimetno di N e K
	int N;
	int K;
	cin >> N >> K;
	
	//creazione albero
	BinTree albero;
	
	//inserimento nodi
	for (int i=0; i < N; ++i){
		int chiave;
		cin >> chiave;
		albero.inserisci(chiave);
	}
	
	albero.calcola_mediani();
	albero.stampa_primi_k_mediani(K);
	
	return 0;
}
``` 

``` cpp
struct Nodo { // 2017-01-11
	int etichetta;
	int conto;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int x) : etichetta(x), conto(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector <int> conti_maggiori_di_k;
	
	void inserisciNodo(Nodo*& u, int x){
		if (u == nullptr){
			u = new Nodo(x);
			return; 
		}
		
		if (x <= u->etichetta)
			inserisciNodo(u->sinistro, x);
		if (x > u->etichetta)
			inserisciNodo(u->destro, x);
	}
	
	//deve per forza essere una visita postOrder per poter vedere prima i figi --> ciò non mi permette di scorrere le etichette in ordine crescente
	int gestioneConto(Nodo* u, int k){ 
		if(u == nullptr)
			return 0;
		
		if (u->sinistro == nullptr && u->destro==nullptr) //nodo foglia
			return 0;
		
		int conto = 0;
		
		// Verifica se figlio sinistro è foglia
		if (u->sinistro && u->sinistro->sinistro == nullptr && u->sinistro->destro == nullptr)
		    conto += punteggioFoglia(u->sinistro->etichetta);
		else
		    conto += gestioneConto(u->sinistro, k);

		// Verifica se figlio destro è foglia
		if (u->destro && u->destro->sinistro == nullptr && u->destro->destro == nullptr)
		    conto += punteggioFoglia(u->destro->etichetta);
		else
		    conto += gestioneConto(u->destro, k);

		u->conto = conto;

		if (conto > k)
		    conti_maggiori_di_k.push_back(u->etichetta);

		return conto;
	}
	
	int punteggioFoglia(int etichetta){
		if(etichetta == 0)
			return 2;
		else if (etichetta % 2 == 0)
			return -1;
		else 
			return 1;
	}
	
public: 
	BinTree(){ 
		radice = nullptr; 
		}
	
	void inserisci (int x) {
		inserisciNodo(radice, x);
	}
	
	void risultato(int k){
		gestioneConto(radice, k);
		
		sort(conti_maggiori_di_k.begin(), conti_maggiori_di_k.end());
		
		int dimensione_vector = conti_maggiori_di_k.size();
		for(int i = 0; i < dimensione_vector; ++i)
			cout << conti_maggiori_di_k[i] << endl;
	
	}

};

int main (){

	//inserimento di N e k
	int N, k;
	cin >> N >> k;
	
	//creazine albero 
	BinTree albero;
	
	//inserimento etichette
	for (int i=0; i < N; ++i){
	int x;
	cin >> x;
	albero.inserisci(x);
	}
	
	albero.risultato(k);
	
	return 0;
}
```

// esercizio con doppia opzione --> destro e sinistro
``` cpp
struct Nodo{ // 2017-01-31
	int etichetta;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int x) : etichetta(x), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	int ZigZag;
	
	void inserisciNodo(Nodo*& u, int x){
		if(u == nullptr){
			u = new Nodo(x);
			return;
		}
		
		if (x <= u->etichetta)
			inserisciNodo(u->sinistro, x);
		if(x > u->etichetta)
			inserisciNodo(u->destro, x);
	}
	
	void contaZigZag(Nodo* u){ //visita postOrder
		if(u == nullptr)
			return;
			
		//visita figli //prima analizziamo tutto il sottoalbero sinistro, poi tutto il destro e infine u
		contaZigZag(u->sinistro);
		contaZigZag(u->destro);
		
		//qui analizziamo il nodo corrente u 
		//caso1: figlio desto unico
		if(u->sinistro == nullptr && u->destro != nullptr) {
			Nodo * y = u->destro;
			if(y->sinistro != nullptr && y->destro == nullptr)
				ZigZag++;
		}
		
		//caso2: figlio sinistro unico
		if (u->sinistro != nullptr && u->destro == nullptr){
			Nodo* y = u->sinistro;
			if(y->sinistro == nullptr && y->destro != nullptr)
				ZigZag++;
		} 
		
		//questi fi si potevano fare anche senza y, bastava scrivere u->destro->sinistro .. però diminuiva la leggibilità
	}
	
public:
	BinTree() {
		radice = nullptr; 
		ZigZag = 0;
	}
	
	void inserisci(int x){
		inserisciNodo(radice, x);
	}
	
	void numeroZigZag(){
		ZigZag = 0;
		contaZigZag(radice);
		cout << ZigZag << endl;
	}
};

int main (){
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.numeroZigZag();
	
	return 0;
}
```

``` cpp
#include <cmath>

struct Nodo { // 2017-02-17
	int valore;
	int altezza_nodo;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v, int h) : valore(v), altezza_nodo(h), sinistro(nullptr), destro(nullptr) {}
};

class BinTree{
	Nodo* radice;
	int livello_con_S_maggiore;
	int score_max;
	int risultato;
	vector<int> numero_nodi_per_livello; //all'indirizzo 'i' c'è scritto il numero di nodi al livello 'i'
	
	void inserisciNodo(Nodo*& u, int v, int h){
		if(u == nullptr){
			u= new Nodo(v,h);
			
			if (h >= (int)numero_nodi_per_livello.size())
				numero_nodi_per_livello.resize(h+1, 0);
			numero_nodi_per_livello[h]++; 
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v, h+1);
		else
			inserisciNodo(u->destro, v, h+1);
	}
	
	void calcolo_score(){ //non serve fare visita perchè si calcola per livello e non per etichetta
		for (int h = 1; h < (int)numero_nodi_per_livello.size(); h++) {
		    	int N_h = numero_nodi_per_livello[h];
		    	double max_h =  pow(2,h-1); // 1 << (h - 1); --> fatto con bit scift --> no errori di arrotondamento
		    	double s_h = ((double)N_h / h) * max_h;

		    	if (s_h > score_max || (s_h == score_max && h > livello_con_S_maggiore)) {
		        	score_max = s_h;
		        	livello_con_S_maggiore = h;
		    	}
		    	/*
		    	if (s_h == score_max){ 
		    		if (H > livello_con_S_maggiore) 
		    			livello_con_S_maggiore = H; } 
		    	else if (s_h > score_max) { 
		    		score_max= s_h; 
		    		livello_con_S_maggiore = H; }
		    	*/
        	}
	}
	
	void calcolo_etichetta_maggiore(Nodo* u){
		if (u == nullptr)
			return;
	
		if(u->altezza_nodo == livello_con_S_maggiore)
			risultato = max(risultato, u->valore);
		
		calcolo_etichetta_maggiore(u->sinistro);
		calcolo_etichetta_maggiore(u->destro);
	}
	
public:
	BinTree () {
		radice= nullptr;
		livello_con_S_maggiore = 0;
		score_max = 0;
		risultato = 0;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v, 1); //in questo es la radice è a livello 1
	}
	
	void soluzione(){
		calcolo_score();
		
		calcolo_etichetta_maggiore(radice);
		
		cout << risultato << endl;;
	}
};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { // 2017-06-08
	int valore;
	int lunghezza_L;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), lunghezza_L(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree{
	Nodo* radice;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
		
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	
	// Ritorna L(u) per il nodo u
	    int verifica_P(Nodo* u){
		
		// casi base
		if(u == nullptr) return 0;

		// Caso foglia: L(leaf) = 0
		if(u->sinistro == nullptr && u->destro == nullptr){
		    u->lunghezza_L = 0;
		    return 0;
		}

		// Calcola ricorsivamente i valori per i figli //visita post order
		int left  = verifica_P(u->sinistro);
		int right = verifica_P(u->destro);

		// Aggiungi +1 se un figlio è foglia e ha parità diversa
		if(u->sinistro != nullptr &&
		   u->sinistro->sinistro == nullptr && u->sinistro->destro == nullptr &&
		   ((u->sinistro->valore % 2) != (u->valore % 2)))
		    	left += 1;
		
		if(u->destro != nullptr &&
		   u->destro->sinistro == nullptr && u->destro->destro == nullptr &&
		   ((u->destro->valore % 2) != (u->valore % 2)))
		    	right += 1;
		

		u->lunghezza_L = left + right; // memorizzo il risultato nel nodo

		return u->lunghezza_L;
		
		
	    }
	
/*
	ALTERNATIVA
	
// Ritorna il numero di foglie-P nel sottoalbero
    int contaFoglieP(Nodo* u, int padreVal) {
        if (u == nullptr) return 0;

        // Se è foglia, verifico la proprietà P
        if (u->sinistro == nullptr && u->destro == nullptr) {
            if ((u->valore % 2 == 0 && padreVal % 2 != 0) || 
                (u->valore % 2 != 0 && padreVal % 2 == 0)) {
                return 1; // soddisfa P
            }
            return 0;
        }

        int left = contaFoglieP(u->sinistro, u->valore);
        int right = contaFoglieP(u->destro, u->valore);

        // salvo L per il nodo u
        u->lunghezza_L = left + right;
        return u->lunghezza_L;
    }
*/	
	void stampa(Nodo* u){ //visita InOrder (ordine crescente)
		if (u == nullptr)
			return;
		
		stampa(u->sinistro);
		cout << u->lunghezza_L << endl;
		stampa(u->destro);
	}
	
public:
	BinTree() {
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v); 
	}
	
	void soluzione(){
		verifica_P(radice);
		stampa(radice);
	}
};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { // 2017-09-12
	int valore;
	bool nodo_P;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), nodo_P(false), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
		
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	int verifica_P(Nodo* u, int k){
		if(u == nullptr)
			return 0; //altezza dell'albero vuoto = 0
			
		int h_sx = verifica_P(u->sinistro, k);
		int h_dx = verifica_P(u->destro, k);
		
		if(u->sinistro == nullptr && u->destro == nullptr)
			u->nodo_P = false;
		else if(abs(h_sx - h_dx) <= (u->valore / k))
			u->nodo_P = true;
		
		return 1 + max(h_sx, h_dx); //ritorno l'altezza del sottoalbero
	}
	
	void stampa(Nodo* u) { //visita InOrder --> crescente
		if (u == nullptr) return;
		
		stampa(u->sinistro);
		
		if (u->nodo_P == true)
		cout << u->valore << endl;
		
		stampa(u->destro);
	}
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(int k){
		verifica_P(radice, k);
		stampa(radice);
	}
};

int main (){
	
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2018-02-01
	int valore;
	int nC;
	int nD;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), nC(0), nD(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
		
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	
	pair<int,int> calcola(Nodo* u, Nodo* padre = nullptr) {
	    if (u == nullptr) 
	    	return {0,0};

	    // Caso foglia
	    if (u->sinistro == nullptr && u->destro == nullptr) {
		if (padre == nullptr) 
			return {0,0}; // radice foglia
		if ((u->valore % 2) == (padre->valore % 2)) 
			return {1,0}; // concorde
		else 
			return {0,1}; // discorde
	    }

	    // Caso nodo interno
	    auto [c1,d1] = calcola(u->sinistro, u); // il ritorno del pair avviene qua
	    auto [c2,d2] = calcola(u->destro, u);

	    u->nC = c1 + c2;
	    u->nD = d1 + d2;

	    return {u->nC, u->nD};
	}
	
	
	void stampa(Nodo* u) { //visita InOrder --> crescente
		if (u == nullptr) return;
		
		stampa(u->sinistro);
		
		if (u->nC - u->nD >= 0)
		cout << u->valore << endl;
		
		stampa(u->destro);
	}
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(){
		calcola(radice);
		stampa(radice);
	}
};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione();
	
	return 0;
}
/*
alternativa senza pair

void calcola(Nodo* u, Nodo* padre, int &nC, int &nD) {
    if (u == nullptr) {
        nC = nD = 0;
        return;
    }

    // caso foglia
    if (u->sinistro == nullptr && u->destro == nullptr) {
        if (padre == nullptr) {
            nC = nD = 0;
        } else if ((u->valore % 2) == (padre->valore % 2)) {
            nC = 1; nD = 0;
        } else {
            nC = 0; nD = 1;
        }
        return;
    }

    int c1,d1,c2,d2;
    calcola(u->sinistro, u, c1, d1);
    calcola(u->destro, u, c2, d2);

    u->nC = c1 + c2;
    u->nD = d1 + d2;

    nC = u->nC;
    nD = u->nD;
}
*/
```

``` cpp
struct Nodo { // 2018-02-19
	int valore;
	int D_x;
	bool pari;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), D_x(-1), pari(false), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector <pair<int,int>> risultati;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
		
			if(v % 2 == 0)
				u->pari = true;
				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	    // u: nodo corrente, prof: profondità attuale -->distanza dalla radice, antenato_pari_max_prof: profondità dell'antenato pari più alto
	    void calcola(Nodo* u, int prof, int antenato_pari_max_prof) { //visita PreOrder
		if(u == nullptr) return;
		
		if(antenato_pari_max_prof != -1) //abbiamo già trovato un antenato pari
		    u->D_x = prof - antenato_pari_max_prof;
		
		// se il nodo stesso è pari, può diventare il nuovo antenato più alto per i discendenti
		int nuovo_antenato_pari_max_prof = antenato_pari_max_prof;
		if(u->pari && antenato_pari_max_prof == -1) //non viene aggiornato se esiste già un antenato pari
		    nuovo_antenato_pari_max_prof = prof;
		
		if(u->D_x > 0)
			risultati.push_back({u->valore, u->D_x});
		
		calcola(u->sinistro, prof + 1, nuovo_antenato_pari_max_prof);
		calcola(u->destro, prof + 1, nuovo_antenato_pari_max_prof);
	    }
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(int k){		
		calcola(radice, 0, -1); //inizio profondità 0, nessun antenato pari
		
		//ordinamento del vector
		sort(risultati.begin(), risultati.end(), 
			[](auto &a, auto &b){
		    	if (a.second != b.second) return a.second > b.second; // D(x) decrescente
		    	return a.first < b.first; // etichetta crescente
		});
		
		for (int i = 0; i < (int)risultati.size() && i < k; i++) {
		    pair<int,int> p = risultati[i];   // estraggo la coppia (etichetta, D_x)
			cout << p.first << "\n";      // stampo l’etichetta
		}

	}
};

int main (){
	
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2018-06-07
	int valore;
	int colore;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v, int c) : valore(v), colore(c), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector<int> lunghezza_colore;
	
	void inserisciNodo(Nodo*& u, int v, int c){
		if(u == nullptr){
			u= new Nodo(v, c);
				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v, c);
		else
			inserisciNodo(u->destro, v, c);
	}
	
	    
	    void calcola(Nodo* u, int colore_riferimento, int lunghezza_riferimento) { //visita PreOrder
		if(u == nullptr) return;
		
		if(colore_riferimento == u->colore){ 
			lunghezza_riferimento++;
		    	lunghezza_colore[colore_riferimento] = max(lunghezza_riferimento, lunghezza_colore[colore_riferimento]);
		    	}
		else{ //colore_riferimento =! u->colore
			colore_riferimento = u->colore;
			lunghezza_riferimento = 1;
			lunghezza_colore[colore_riferimento] = max(lunghezza_riferimento, lunghezza_colore[colore_riferimento]);
		}
				
		calcola(u->sinistro, colore_riferimento, lunghezza_riferimento);
		calcola(u->destro, colore_riferimento, lunghezza_riferimento);
	    }
    
	
public:
	BinTree( int C){
		radice = nullptr;
		lunghezza_colore.resize(C); //riempie il vector di 0
	}
	
	void inserisci(int v, int c){
		inserisciNodo(radice, v, c);
	}
	
	void soluzione(int k){		
		calcola(radice, -1, 0);  //-1 è in riferimento al colore // 0 è la lunghezza di riferimento di partenza
		
		for(int i = 0; i < (int)lunghezza_colore.size(); ++i)
	    		cout << lunghezza_colore[i] << endl;

	}

};

int main (){
	
	int N, C;
	cin >> N >> C;
	
	BinTree albero(C);
	
	for(int i=0; i < N; ++i){
		int x, c;
		cin >> x >> c;
		albero.inserisci(x, c);
	}
	
	albero.soluzione(C);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2018-06-28
	int valore;
	string stringa;
	
	int distanza; //distanza tra il nodo e la radice --> simil altezza
	int num_foglie; //numero di foglie appartenenti al sottoalbero radicato in x
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v, string s, int d) : valore(v), stringa(s), distanza(d), num_foglie(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector<string> nodi_P;
	
	void inserisciNodo(Nodo*& u, int v, string s, int d){
		if(u == nullptr){
			u= new Nodo(v, s, d);				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v, s, d+1);
		else
			inserisciNodo(u->destro, v, s, d+1);
	}
	
	int verifica_P(Nodo* u) { //visita PostOrder
		if(u == nullptr) return 0;
		
		if(u->sinistro == nullptr && u->destro == nullptr){
		    u->num_foglie = 0;
		    if (u->num_foglie == u->distanza) //necessario quando ho un solo nodo
			nodi_P.push_back(u->stringa);
		    return 0;
		}

		int left = verifica_P(u->sinistro);
		int right = verifica_P(u->destro);
		
		if (u->sinistro != nullptr &&
			u->sinistro->sinistro == nullptr && u->sinistro->destro == nullptr)
			left++;
		if(u->destro != nullptr &&
			u->destro->sinistro == nullptr && u->destro->destro == nullptr)
			right++;
			
		//verifica proprietà 
		u->num_foglie = left+right;
		if(u->num_foglie == u->distanza)
			nodi_P.push_back(u->stringa);
		
		return u->num_foglie;
	    }
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v, string s){
		inserisciNodo(radice, v, s, 0); //la radice è a distanza 0 da se stessa
	}
	
	void soluzione(){		
		verifica_P(radice); 
		
		//ordinamento del vector
		sort(nodi_P.begin(), nodi_P.end() ); // se non metto condizione ordina in ordine crescente
		
		for (int i = 0; i < nodi_P.size(); i++) { //non serve (int)nodi_P.size()
			cout << nodi_P[i] << endl;      // stampo la stringa
		}
		
	 	return;
	}

};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		string s;
		cin >> x >> s;
		albero.inserisci(x, s);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { // 2018-09-13
	int valore;
	bool pari;
	int D;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), pari(v % 2 == 0), D(-1), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector<int> valori_D; 
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	pair<int,int> calcola_D(Nodo* u) { //visita PostOrder //distanza dal pari, distanza dispari
		if (u == nullptr) return {-1, -1};
		
		//caso foglia
		if(u->sinistro == nullptr && u->destro == nullptr){
			if ( u->pari == true )
				return {0, -1};
			else // u è dispari
				return {-1, 0};
		}
		//caso nodo interno
		auto [dist_pari_dx, dist_disp_dx] = calcola_D(u->destro);
		auto [dist_pari_sx, dist_disp_sx] = calcola_D(u->sinistro);
		
		int dist_pari;
		if(dist_pari_dx == -1 && dist_pari_sx == -1) dist_pari = -1;
		else if (dist_pari_dx == -1) dist_pari = dist_pari_sx +1;
		else if (dist_pari_sx == -1) dist_pari = dist_pari_dx +1;
		else dist_pari= max(dist_pari_dx, dist_pari_sx) +1;
		
		int dist_disp;
		if(dist_disp_dx == -1 && dist_disp_sx == -1) dist_disp = -1;
		else if (dist_disp_dx == -1) dist_disp = dist_disp_sx +1;
		else if (dist_disp_sx == -1) dist_disp = dist_disp_dx +1;
		else dist_disp = max(dist_disp_dx, dist_disp_sx) +1;
		
		// calcolo D
		if(u->pari == true)
			u->D = dist_pari;
		else // se dispari
			u->D = dist_disp;
			
		valori_D.push_back(u->D);
		
		return {dist_pari, dist_disp}; //restituisco entrambi perchè non so se padre è pari o dispiari
	}
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(){		
		calcola_D(radice); 
		
		//ordinamento del vector
		sort(valori_D.begin(), valori_D.end(), 
			[](int a, int b){ return a > b;}); // ordine decrescente 
		
		for (int i = 0, k= -1; i < valori_D.size(); i++, k++) { 
			if (k == -1)
				cout << valori_D[i] << endl;
			else if (valori_D[i] != valori_D[k])
				cout << valori_D[i] << endl;
		}
	 	return;
	}

};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x ;
		albero.inserisci(x);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { // 2019-01-31
	int valore;
	
	int eq; //numero di eq nel sottoalbero, x incluso
	int neq;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), eq(0), neq(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	int max_f;
	int etichetta_max_f;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	pair<int,int> calcola_f(Nodo* u){ //visita postorder //eq, neq
	
		if(u == nullptr)
			return {0,0};
		if(u->sinistro == nullptr && u->destro== nullptr){
			int f = 0;
			if(etichetta_max_f == -1){ //prima volta che calcolo f
				etichetta_max_f = u->valore;
				max_f = 0;
			}
			else {
				if(f > max_f){
					max_f = f;
					etichetta_max_f = u->valore;
				}
				else if (f == max_f)
					etichetta_max_f = min(u->valore, etichetta_max_f);
			}
			return {0,0};
		}
		
		auto [eq_sx, neq_sx] = calcola_f(u->sinistro);
		auto[eq_dx, neq_dx] = calcola_f(u->destro);
		
		u->eq = eq_dx + eq_sx;
		u->neq = neq_dx + neq_sx;
		
		//nodo eq
		if(u->sinistro != nullptr && u->destro != nullptr &&
			u->sinistro->sinistro == nullptr && u->sinistro->destro == nullptr && //figlio sx foglia
			u->destro->sinistro == nullptr && u->destro->destro == nullptr) //figlio dx foglia
			u->eq++;
			
		// nodo neq
		if((u->sinistro != nullptr || u->destro != nullptr) &&
			(u->sinistro == nullptr ^ u->destro == nullptr)) { // or esclusivo
			if (u->sinistro != nullptr){ //il figlio è il sx
				if(u->sinistro->sinistro == nullptr && u->sinistro->destro==nullptr)
				u->neq++;
				}
			else{ //il figlio è il dx
				if(u->destro->sinistro == nullptr && u->destro->destro == nullptr)
				u->neq++;
				}
		}
		
		//verifica di f
		int f = u->neq - u->eq;
		if(f > max_f){
			max_f = f;
			etichetta_max_f = u->valore;
		}
		if (f == max_f)
			etichetta_max_f = min(u->valore, etichetta_max_f);
			
		return {u->eq, u->neq};
	}
	
public:
	BinTree(){
		radice = nullptr;
		max_f = 0;
		etichetta_max_f = -1;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(){		
		calcola_f(radice); 
		
		//stampa
		cout << etichetta_max_f << endl;
		cout << max_f << endl;
	 	return;
	}
};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x ;
		albero.inserisci(x);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { // 2019-02-18
	int valore;
	int massa;
	int distanza;
	
	int peso;
	int carico;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v, int m, int d) : valore(v), massa(m), distanza(d), carico(0), peso(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector<int> carichi;
	
	void inserisciNodo(Nodo*& u, int v, int m, int d){
		if(u == nullptr){
			u= new Nodo(v, m, d);
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v, m, d+1);
		else
			inserisciNodo(u->destro, v, m, d+1);
	}
	
	pair<int,int> calcola(Nodo* u){ //visita postOrder //peso, carico
		if(u == nullptr)
			return{0,0};
		
		if(u->sinistro == nullptr && u->destro == nullptr){
			u->carico = 0;
			u->peso = (u->massa * 2) - u->distanza;
			
			carichi.push_back(u->carico);
		
			return {u->peso,0};
		}
		
		auto[peso_sx, carico_sx] = calcola(u->sinistro);
		auto[peso_dx, carico_dx] = calcola(u->destro);
		
		u->carico = carico_sx + carico_dx + peso_sx + peso_dx;
		
		carichi.push_back(u->carico);
		
		//siamo già nel caso in cui u non è foglia
		u->peso = u->massa - u->distanza;
		return {u->peso, u->carico};
	}
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v, int x){
		inserisciNodo(radice, v, x, 0);
	}
	
	void soluzione(int k){		
		calcola(radice); 
		
		//stampa
		sort(carichi.begin(), carichi.end(), 
			[](int a, int b){return a>b;});
		
		for(int i=0; i<k && i<carichi.size(); i++){
			cout << carichi[i] << endl;
		}
	}
};

int main (){
	
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x, m;
		cin >> x >> m;
		albero.inserisci(x, m);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2019-06-06
	int valore;
	int h_x;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), h_x(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	int H_radice;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	int calcola(Nodo* u){ //visita postOrder 
		if(u == nullptr)
			return 0;
		
		if(u->sinistro == nullptr && u->destro == nullptr){
			u->h_x = 1;
			H_radice = max(u->h_x, H_radice);
			return 1;
		}
		
		int h_x_sx = calcola(u->sinistro);
		int h_x_dx= calcola(u->destro);
		
		u->h_x = max(h_x_sx, h_x_dx) +1;
		
		H_radice = max(u->h_x, H_radice);
		
		return u->h_x;
	}
	
	void verifica_e_stampa(Nodo* u, int& k){ //ordine crescente --> visita InOrder
		if(u == nullptr || k==0)
			return;
			
		verifica_e_stampa(u->sinistro, k);
		
		if(u->h_x == H_radice/2 && k>0){
			cout << u->valore << endl;
			k--;
			if(k == 0) return;
			}
		
		verifica_e_stampa(u->destro, k);
	}
	
public:
	BinTree(){
		radice = nullptr;
		H_radice=0;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(int k){		
		calcola(radice);
		//potevo fare H_radice = calcola(radice) --> senza dover incrementare volta per volta in calcola anche H_radice 
		verifica_e_stampa(radice, k);
	}
};

int main (){
	
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { //2019-06-27
    int valore;
    char tipo;
    int g;
    
    Nodo* sinistro;
    Nodo* destro;
    
    Nodo(int v, char t) : valore(v), tipo(t), g(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;
    
    void inserisciNodo(Nodo*& u, int v, char t) {
        if(u == nullptr) {
            u = new Nodo(v, t);
            return;
        }
        
        if(v <= u->valore)
            inserisciNodo(u->sinistro, v, t);
        else
            inserisciNodo(u->destro, v, t);
    }
    
    /*
    stato:
    0 -> partito dal server (in cerca di filtro)
    1 -> arrivato a un filtro (in cerca di client)
    2 -> client raggiunto (trovato client)
    */
    
    int calcola(Nodo* u, int stato) {
        if(u == nullptr) return 0;
        
        int add = 0; //conta i client raggiunti da questo nodo
        int nuovo_stato = stato; //nuovo_stato è lo stato aggiornato dopo aver processato questo nodo
        
        if(u->tipo == 'C') {
            if(stato != 0) { //pronti per raggiungere il client
                nuovo_stato = 2;
                add = 1; //conta questo client
            }
        }
        else if(u->tipo == 'S') {
            nuovo_stato = 0; //inizio nuovo cammino
        }
        else if(u->tipo == 'F') {
            if(stato == 0) {
                nuovo_stato = 1;
            }
        }
        
        // visita ricorsiva dei figli con nuovo stato
        int sx = calcola(u->sinistro, nuovo_stato);
        int dx = calcola(u->destro, nuovo_stato);
        
        
        if(u->tipo == 'S' && nuovo_stato == 0) {
            u->g = sx + dx; //conta i cammini completati dai figli
            return 0; //ritorno 0 perchè i cammini non possono attraversare più server
        }
        
        return sx + dx + add;
    }
    
    void stampaServer(Nodo* u) {
        if(u == nullptr) return;
        
        stampaServer(u->sinistro);
        if(u->tipo == 'S') 
        	cout << u->valore << " " << u->g << endl;
        stampaServer(u->destro);
    }
    
public:
    BinTree() : radice(nullptr) {}
    
    void inserisci(int v, char t) {
        inserisciNodo(radice, v, t);
    }
    
    void soluzione() {
        calcola(radice, 0);
        stampaServer(radice);
    }
};

int main() {
    int N;
    cin >> N;
    
    BinTree albero;
    
    for(int i = 0; i < N; i++) {
        int id;
        char tipo;
        cin >> id >> tipo;
        albero.inserisci(id, tipo);
    }
    
    albero.soluzione();
    
    return 0;
}
```

``` cpp
#include <cmath>

struct Nodo { // 2022-05-27
	int valore;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), sinistro(nullptr), destro(nullptr) {}
};

class BinTree{
	Nodo* radice;
	bool parametro;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
		
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	

	int verifica(Nodo* u){ //visita postOrder
		if(u == nullptr) return 0;

	//la foglia ha altezza uguale a 1

		int left  = verifica(u->sinistro);
		int right = verifica(u->destro);

		if( abs(left-right) > 1)
			parametro = false;

		return 1 + max(left, right); //altezza nodo corrente
	    }
	

	
public:
	BinTree() {
		radice = nullptr;
		parametro = true;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v); 
	}
	
	void soluzione(){
		verifica(radice);
		
		if (parametro == true)
			cout << "ok" << endl;
		else 
			cout << "no" << endl;
	}
};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { //2023-01-19
    int valore;
    int altezza;

    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v, int h) : valore(v), altezza(h), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;
    vector<pair<int,int>> pesi_livelli; //(peso, livello)

    void inserisciNodo(Nodo*& u, int v, int h) {
        if (u == nullptr) {
            u = new Nodo(v, h);
            
            if(h >= pesi_livelli.size())
            	pesi_livelli.resize(h+1, {0, h});
            	
            	
            pesi_livelli[h].first = pesi_livelli[h].first + v;
            pesi_livelli[h].second = h;
            
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, h+1);
        else                 
        	inserisciNodo(u->destro,  v, h+1);
    }


public:
    	BinTree(){
		radice=nullptr;
	}

	void inserisci(int v) { 
    		inserisciNodo(radice, v, 0); 
    	}

	void soluzione(int k) { 
		// Ordina per peso decrescente, in caso di parità per livello decrescente
		sort(pesi_livelli.begin(), pesi_livelli.end(), 
		    	[](auto a, auto b) {
				if (a.first != b.first) 
				    return a.first > b.first;
				return a.second > b.second;
		    });
		
		// Prendi i primi k elementi (coppie peso-livello)
		vector<pair<int,int>> primi_k;
		for (int i = 0; i < k && i < pesi_livelli.size(); i++) {
		    primi_k.push_back(pesi_livelli[i]);
		}
		
		// Ordina i livelli selezionati per livello crescente
		sort(primi_k.begin(), primi_k.end(),
			[](auto a, auto b){ return a.second < b.second;}
			);
		
		// Stampa i pesi in ordine di livello crescente
		for (int i=0; i < primi_k.size(); i++) {
		    cout << primi_k[i].first << endl;
		}
	}
	
/*
ALTERNATIVA PER LA FUNZIONE SOLUZIONE --> UTILIZZO DEL RESIZE PER ACCORCIARE IL VECTOR --> non serve creare primi_k

	void soluzione(int k) { 
	    sort(pesi_livelli.begin(), pesi_livelli.end(), 
		 [](auto a, auto b) {
		     if (a.first != b.first) 
		         return a.first > b.first;
		     return a.second > b.second;
		 });

-->	    if ((int)pesi_livelli.size() > k)
		pesi_livelli.resize(k); //se la dimensione del vector è maggiore di k taglia --> se è minore aggiunge

	    sort(pesi_livelli.begin(), pesi_livelli.end(),
		 [](auto a, auto b){ return a.second < b.second;}
	    );

	    for (int i=0; i<pesi_livelli.size(); i++)
		cout << pesi_livelli[i].first << endl;
	}
*/
};

int main() {
	// inserimento di N
	int N;
	cin >> N;
	
	//inserimento di k
	int k;
	cin >> k; 
	
	//creazione albero
	BinTree albero;
	
	//inserimento dei noi
	for (int i=0; i<N; ++i){
		int valore;
		cin >> valore;
		albero.inserisci(valore);
	}
	
	albero.soluzione(k);
	
	return 0;
}
```

``` cpp
struct Nodo { //2023-02-03
	int valore; //id
	string nome;
	int altezza;
	int discendenti;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v, string n, int h) : valore(v), nome(n), altezza(h), discendenti(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector<string> risultati;
	
	void inserisciNodo(Nodo*& u, int v, string n, int h){
		if(u == nullptr){
			u= new Nodo(v, n, h);
				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v, n, h+1);
		else
			inserisciNodo(u->destro, v, n, h+1);
	}
	
	int calcola(Nodo* u) { //visita PostOrder
		if(u == nullptr) 
			return 0;
		
		if (u->sinistro== nullptr && u->destro==nullptr){
			u->discendenti = 0;
			if(u->discendenti == u->altezza)
				risultati.push_back(u->nome);
			return 1;
			}
		
		int left = calcola(u->sinistro);
		int right = calcola(u->destro);
		
		u->discendenti = left + right;
		
		if(u->altezza == u->discendenti)
			risultati.push_back(u->nome);
		
		return u->discendenti+1;
		
	    }
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v, string n){
		inserisciNodo(radice, v, n, 0); //altezza radice = 0;
	}
	
	void soluzione(){		
		calcola(radice); 
		
		sort(risultati.begin(), risultati.end() );
		
		for (int i = 0; i < risultati.size(); i++) {
			cout << risultati[i] << endl;
		}

	}
};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		string n;
		cin >> x >> n;
		albero.inserisci(x, n);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { //2023-02-23
	int valore; //id
	int d_x; //altezza considerando radice ad altezza 0
	int l_x; //massima distanza tra x e le foglie del suo sottoalbero
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v, int d) : valore(v), d_x(d), l_x(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector<int> risultati;
	
	void inserisciNodo(Nodo*& u, int v, int d){
		if(u == nullptr){
			u= new Nodo(v, d);
				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v, d+1);
		else
			inserisciNodo(u->destro, v, d+1);
	}
	
	int calcola(Nodo* u) { //visita PostOrder
		if(u == nullptr) 
			return 0;
		
		if (u->sinistro== nullptr && u->destro==nullptr){
			u->l_x = 0;
			if(abs(u->l_x - u->d_x) <= 1)
				risultati.push_back(u->valore);
			return 1;
			}
		
		int left = calcola(u->sinistro);
		int right = calcola(u->destro);
		
		u->l_x = max(left, right);
		
		if(abs(u->l_x - u->d_x) <= 1)
			risultati.push_back(u->valore);
		
		return u->l_x+1;
		
	    }
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v, 0); //altezza radice = 0;
	}
	
	void soluzione(int k){		
		calcola(radice); 
		
		sort(risultati.begin(), risultati.end() );
		
		for (int i = 0; i < risultati.size() && i < k; i++) {
			cout << risultati[i] << endl;
		}

	}
};

int main (){
	
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x ;
		albero.inserisci(x);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2023-04-19
	int valore; //id
	int punti;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), punti(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector<pair<int,int>> risultati; // (etichetta, punti)
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	int calcola(Nodo* u) { //visita PostOrder
		if(u == nullptr) 
			return 0;
		
		if (u->sinistro== nullptr && u->destro==nullptr){
		
			risultati.push_back({u->valore, u->punti});
			
			if (u->valore == 0)
				return 2; //conto_figli = conto_figli +2;
			else if (u->valore % 2 == 0)
				return -1; //conto_figli--;
			else 
				return +1; //conto_figli++;
			}
		
		int left = calcola(u->sinistro);
		int right = calcola(u->destro);
		
		u->punti = left+right;
		risultati.push_back({u->valore, u->punti});
		
		return u->punti;
		
	    }
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v); 
	}
	
	void soluzione(int k){		
		calcola(radice); 
		
		sort(risultati.begin(), risultati.end(),
			[](auto a, auto b){
			if(a.second != b.second) return a.second < b.second;
			return a.first > b.first;
			}
		);
		
		for (int i = 0; i < risultati.size() && i < k; i++) {
			cout << risultati[i].first << endl;
		}

	}
};

int main (){
	
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x ;
		albero.inserisci(x);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2023-06-15
	int valore;
	int num_figli_unici;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), num_figli_unici(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	int calcola(Nodo* u) { //visita PostOrder
		if(u == nullptr) 
			return 0;
		
		if (u->sinistro== nullptr && u->destro==nullptr){
			u->num_figli_unici = 0;
			return 0;
			}
			
		int contatore= 0;
		if(u->destro == nullptr && u->sinistro != nullptr){
			if(u->sinistro->sinistro == nullptr && u->sinistro->destro == nullptr)
			contatore++;
		} //ha figlio sinistro che è foglia
		
		if(u->sinistro == nullptr && u->destro != nullptr){
			if(u->destro->sinistro == nullptr && u->destro->destro == nullptr)
			contatore++;
		}
			
		if(contatore == 1){
			u->num_figli_unici = 1;
			return 1;
			}
			
		int left = calcola(u->sinistro);
		int right = calcola(u->destro);
			
		u->num_figli_unici = left+right;
		
		return u->num_figli_unici;
		
	    }
	    
	void stampa (Nodo* u, int inferiore, int superiore){ // ordine crescente --> visita InOrder
		if (u == nullptr) return;
		
		stampa(u->sinistro, inferiore, superiore);
		
		if (u->num_figli_unici >= inferiore && u->num_figli_unici <= superiore)
			cout << u->valore << endl;
			
		stampa(u->destro, inferiore, superiore);
	}
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v); 
	}
	
	void soluzione(int A, int B){		
		calcola(radice); 
		
		int inferiore = min(A, B);
		int superiore = max(A,B);
		
		stampa(radice, inferiore, superiore);
	}
};

int main (){
	
	int N, A, B;
	cin >> N >> A >> B;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x ;
		albero.inserisci(x);
	}
	
	albero.soluzione(A, B);
	
	return 0;
}
```

``` cpp
struct Nodo { //2023-07-06
	int valore;
	int nC;
	int nD;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), nC(0), nD(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
		
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	pair<int,int> calcola(Nodo* u, Nodo* padre = nullptr) {
	    if (u == nullptr) 
	    	return {0,0};

	    // Caso foglia --> le foglie hanno valore doppio
	    if (u->sinistro == nullptr && u->destro == nullptr) {
		if (padre == nullptr) 
			return {0,0}; // radice foglia
		if ((u->valore % 2) == (padre->valore % 2)) 
			return {2,0}; // concorde
		else 
			return {0,2}; // discorde
	    }

	    // Caso nodo interno
	    auto [c1,d1] = calcola(u->sinistro, u); 
	    auto [c2,d2] = calcola(u->destro, u);

	    u->nC = c1 + c2;
	    u->nD = d1 + d2;
	    
	    //non foglia
	    if (padre == nullptr) 
			return {u->nC, u->nD}; //  foglia
	    if ((u->valore % 2) == (padre->valore % 2)) 
			return {u->nC + 1, u->nD}; // concorde
	    else 
			return {u->nC, u->nD +1}; // discorde
	}
	
	
	void stampa(Nodo* u) { //visita InOrder --> crescente
		if (u == nullptr) return;
		
		stampa(u->sinistro);
		
		if (u->nC > u->nD)
		cout << u->valore << endl;
		
		stampa(u->destro);
	}
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(){
		calcola(radice);
		stampa(radice);
	}

};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { //2023-07-18
	int valore;
	char colore;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v, char c) : valore(v), colore(c), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector<int> nodi_accerchiati;
	
	void inserisciNodo(Nodo*& u, int v, char c){
		if(u == nullptr){
			u= new Nodo(v, c);
				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v, c);
		else
			inserisciNodo(u->destro, v, c);
	}
	
/* 
	    void calcola(Nodo* u) { // visita InOrder
		if(u == nullptr) 
			return;
		
		if(u->sinistro == nullptr && u->destro == nullptr) //foglia
			return;
			
		if((u->sinistro != nullptr && u->sinistro->sinistro == nullptr && u->sinistro->destro == nullptr) && (u->destro != nullptr && u->destro->sinistro==nullptr && u->destro->destro == nullptr))
			return;
			
		calcola(u->sinistro);
		 
		 char colore_riferimento = u->colore;
		 if((u->sinistro != nullptr && u->sinistro->sinistro!= nullptr && u->sinistro->destro!=nullptr && u->sinistro->sinistro->colore == colore_riferimento && u->sinistro->destro->colore == colore_riferimento) || (u->destro != nullptr && u->destro->sinistro!=nullptr && u->destro->destro != nullptr && u->destro->sinistro->colore == colore_riferimento && u->destro->destro->colore == colore_riferimento))
		 nodi_accerchiati.push_back(u->valore);
		 
		 calcola(u->destro);	
	    }
*/
	// verisone più elegante
    bool isAccerchiato(Nodo* u, char c) {
		return u != nullptr &&
		       u->sinistro != nullptr &&
		       u->destro != nullptr &&
		       u->sinistro->colore == c &&
		       u->destro->colore == c;
	    }

	void calcola(Nodo* u) {
		if (u == nullptr) return;
		calcola(u->sinistro);

		if ((u->sinistro && isAccerchiato(u->sinistro, u->colore)) ||
		    (u->destro   && isAccerchiato(u->destro, u->colore))) {
		    nodi_accerchiati.push_back(u->valore);
		}

		calcola(u->destro);
	    }
	
public:
	BinTree( ){
		radice = nullptr; 
	}
	
	void inserisci(int v, char c){
		inserisciNodo(radice, v, c);
	}
	
	void soluzione(){		
		calcola(radice); 
		
		for(int i = 0; i < nodi_accerchiati.size(); ++i)
	    		cout << nodi_accerchiati[i] << endl;
	}

};

int main (){
	
	int N;
	cin >> N;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		char c;
		cin >> x >> c;
		albero.inserisci(x, c);
	}
	
	albero.soluzione();
	
	return 0;
}
```

``` cpp
struct Nodo { // 2023-09-20
	int valore;
	int peso;
	int S; // somma dei pesi dei discendenti con distanza <= 2
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int x, int p) : valore(x), peso(p), S(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector <pair<int,int>> nodi_P; // (valore, peso)
	
	void inserisciNodo(Nodo*& u, int x, int p){
		if (u == nullptr){
			u = new Nodo(x, p);
			return; 
		}
		
		if (x <= u->valore)
			inserisciNodo(u->sinistro, x, p);
		if (x > u->valore)
			inserisciNodo(u->destro, x, p);
	}

	void verifica_P(Nodo* u, int k){ //visita postOrder
		if(u == nullptr)
			return;
		
		if (u->sinistro == nullptr && u->destro == nullptr) //nodo foglia
			return;
			
		verifica_P(u->sinistro, k);
		verifica_P(u->destro, k);
		
		if (u->sinistro != nullptr) {
			u->S += u->sinistro->peso;
			if (u->sinistro->sinistro != nullptr) u->S += u->sinistro->sinistro->peso;
			if (u->sinistro->destro != nullptr)   u->S += u->sinistro->destro->peso;
		}
	    	if (u->destro != nullptr) {
			u->S += u->destro->peso;
			if (u->destro->sinistro != nullptr) u->S += u->destro->sinistro->peso;
			if (u->destro->destro != nullptr)   u->S += u->destro->destro->peso;
	    	}
		
		if( (u->peso * k) < u->S)
			nodi_P.push_back({u->valore, u->peso});

	}

	
public: 
	BinTree(){ 
		radice = nullptr; 
		}
	
	void inserisci (int x, int p) {
		inserisciNodo(radice, x, p);
	}
	
	void soluzione(int k){
		verifica_P(radice, k);
		
		sort(nodi_P.begin(), nodi_P.end(), 
			[](auto a, auto b){
				if(a.second != b.second) return a.second > b.second;
				return a.first > b.first;
			}
		);
				
		if(nodi_P.size() > 0)
			cout << nodi_P[0].first << endl;
	}

};

int main (){

	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for (int i=0; i < N; ++i){
		int x, p;
		cin >> x >> p;
		albero.inserisci(x, p);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2024-01-09
	int valore;
	int D_x;
	bool pari;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), D_x(-1), pari(false), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	vector <pair<int,int>> risultati; //(etichetta, D_(x))
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
		
			if(v % 2 == 0 || v == 0)
				u->pari = true;
				
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	    // u: nodo corrente, prof: profondità attuale -->distanza dalla radice, antenato_pari_max_prof: profondità dell'ultimo antenato pari incontrato
	    void calcola(Nodo* u, int prof, int antenato_pari_max_prof) { //visita PreOrder
		if(u == nullptr) return;
		
		if(antenato_pari_max_prof != -1) //abbiamo già trovato un antenato pari
		    u->D_x = prof - antenato_pari_max_prof;
		
		// se il nodo stesso è pari, può diventare il nuovo antenato pari per i discendenti
		int nuovo_antenato_pari_max_prof = antenato_pari_max_prof;
		if(u->pari == true )
		    nuovo_antenato_pari_max_prof = prof;
		
		if(u->D_x > 0)
			risultati.push_back({u->valore, u->D_x});
		
		calcola(u->sinistro, prof + 1, nuovo_antenato_pari_max_prof);
		calcola(u->destro, prof + 1, nuovo_antenato_pari_max_prof);
	    }
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(int k){		
		calcola(radice, 0, -1); //inizio profondità 0, nessun antenato pari
		
		//ordinamento del vector
		sort(risultati.begin(), risultati.end(), 
			[](auto &a, auto &b){
		    	if (a.second != b.second) return a.second > b.second; // D(x) decrescente
		    	return a.first > b.first; // etichetta decresente
		});
		
		for (int i = 0; i < risultati.size() && i < k; i++) {
			cout << risultati[i].first << "\n";   
		}
	}
};

int main (){
	
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2024-01-26
	int valore;
	bool nodo_P;
	
	Nodo* sinistro;
	Nodo* destro;
	
	Nodo(int v) : valore(v), nodo_P(false), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
	Nodo* radice;
	
	void inserisciNodo(Nodo*& u, int v){
		if(u == nullptr){
			u= new Nodo(v);
			return;
		}
		
		if(v <= u->valore)
			inserisciNodo(u->sinistro, v);
		else
			inserisciNodo(u->destro, v);
	}
	
	int verifica_P(Nodo* u, int k){
		if(u == nullptr)
			return 0; //altezza dell'albero vuoto = 0
			
		int h_sx = verifica_P(u->sinistro, k);
		int h_dx = verifica_P(u->destro, k);
		
		if(u->sinistro == nullptr && u->destro == nullptr)
			u->nodo_P = true;
		else if(abs(h_sx - h_dx) < (k))
			u->nodo_P = true;
		
		return 1 + max(h_sx, h_dx); //ritorno l'altezza del sottoalbero
	}
	
	void stampa(Nodo* u) { //visita InOrder --> crescente
		if (u == nullptr) return;
		
		stampa(u->sinistro);
		
		if (u->nodo_P == true)
			cout << u->valore << endl;
		
		stampa(u->destro);
	}
	
public:
	BinTree(){
		radice = nullptr;
	}
	
	void inserisci(int v){
		inserisciNodo(radice, v);
	}
	
	void soluzione(int k){
		verifica_P(radice, k);
		stampa(radice);
	}
};

int main (){
	
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for(int i=0; i < N; ++i){
		int x;
		cin >> x;
		albero.inserisci(x);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2024-02-13
    int valore;
    
    int D;              // somma delle etichette el sottoalbero destro
    int S;              // somma delle etichette el sottoalbero sinistro
    int P; 		// somma antenati
    
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v) : valore(v), D(0), S(0), P(0), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;
    vector<pair<int, int>> risultati; //(etichetta, F)

    void inserisciNodo(Nodo*& u, int v, int antenati) {
        if (u == nullptr) { 
            u = new Nodo(v);
            u->P = antenati;
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, antenati+u->valore);
        else                 
        	inserisciNodo(u->destro,  v, antenati+ u->valore);
    }

    int calcolaSD(Nodo* u) { //postOrder
        if (u == nullptr) 
            return 0;

        int Sx = calcolaSD(u->sinistro); // S(u) //visita sottoalbero sinistro
        int Dx = calcolaSD(u->destro); // D(u) //visita sottoalbero destro
        
        u->S = Sx; 
        u->D = Dx;
        
        int F = u->P - u->S - u->D; 
        risultati.push_back({u->valore, F});
	
	return Sx + Dx + u->valore; //restituisco la somma totale
    }


public:
    BinTree() : radice(nullptr) {}

    void inserisci(int v) { 
    	inserisciNodo(radice, v, 0); 
    	}

    void soluzione(int k) { 
        calcolaSD(radice);
        
        sort (risultati.begin(), risultati.end(),
        	[](auto a, auto b){
        	if(a.second != b.second) return a.second < b.second;
        	//if(a.second == b.second) 
        	return a.first < b.first; 
        	});
        	
        for(int i= 0; i<risultati.size() && i < k; i++)
        	cout << risultati[i].first << endl;
    }
};

int main() {
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for (int i=0; i<N; ++i){
		int valore;
		cin >> valore;
		albero.inserisci(valore);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { // 2024-06-04
    int valore;
    
    int nDsx;		// num nodi con etichetta dispari nel sottoalbero sinistro di x
    int nPdx;		// num nodi con etichetta pari nel sottoalbero destro di x
    int lev;		//altezza
    
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v, int h) : valore(v), nDsx(0), nPdx(0), lev(h), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;
    vector<int> risultati; 

    void inserisciNodo(Nodo*& u, int v, int h) {
        if (u == nullptr) { 
            u = new Nodo(v, h);
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, h+1);
        else                 
        	inserisciNodo(u->destro,  v, h+1);
    }

    pair<int,int> calcola(Nodo* u) { //postOrder
    					//(nDsx, nPdx)
        if (u == nullptr) 
            return {0,0};
       
        	
        auto [num_disp_figliodx, num_pari_figliodx] = calcola(u->destro);
        auto [num_disp_figliosx, num_pari_figliosx] = calcola(u->sinistro);
        
        u->nDsx = num_disp_figliosx; 
        u->nPdx = num_pari_figliodx;
        
        if(abs(u->nDsx - u->nPdx) <= u->lev)
        	risultati.push_back(u->valore);
        	
	if (u->valore % 2 == 0)
		return {u->nDsx + num_disp_figliodx, u->nPdx + 1 + num_pari_figliosx};
	return {u->nDsx +1 + num_disp_figliodx, u->nPdx + num_pari_figliosx};
    }


public:
    BinTree() : radice(nullptr) {}

    void inserisci(int v) { 
    	inserisciNodo(radice, v, 0); 
    	}

    void soluzione(int k) { 
        calcola(radice);
        
        sort (risultati.begin(), risultati.end()); //oridine cerscente
        	
        for(int i= 0; i<risultati.size() && i < k; i++)
        	cout << risultati[i] << endl;
    }
      
};

int main() {
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for (int i=0; i<N; ++i){
		int valore;
		cin >> valore;
		albero.inserisci(valore);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { //2024-07-16
    int valore;
    int W; 		// consumo/ rifornimento per quella tappa 
    
    int h;		//altezza
    
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v, int W, int h) : valore(v), W(W), h(h), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;
    int ID; //id della destinazione
    int lunghezza; //numero delle tappe intermedie del percorso 

    void inserisciNodo(Nodo*& u, int v, int W, int h) {
        if (u == nullptr) { 
            u = new Nodo(v, W, h);
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, W, h+1);
        else                 
        	inserisciNodo(u->destro,  v, W, h+1);
    }

    void calcola(Nodo* u, int C) { //preOrder //C= carburante attuale --> passato per valore non riferimento --> quindi nellla ricorsine no problem quando campio percorso
        if (u == nullptr) 
            return;
        
        C = C + u->W;
        if(C < 0)
        	return;
        	
        int lungh = u->h -1;
        	
        if(u->sinistro == nullptr && u->destro== nullptr){
        	if(lungh > lunghezza){
        		lunghezza= lungh;
        		ID = u->valore;
        	}
        	else if(lungh == lunghezza && u->valore < ID)
        		ID = u->valore;
        	
        	return;
        	
        }
        
        calcola(u->sinistro, C);
        calcola(u->destro, C);
    }

public:
    BinTree() {
    	radice = nullptr;
    	ID = -1;
    	lunghezza = -1; 
    }

    void inserisci(int v, int W) { 
    	inserisciNodo(radice, v, W, 0); 
    	}

    void soluzione(int C) { 
        calcola(radice, C);
        
        cout << ID << " " << lunghezza << endl;
    } 
};

int main() {
	int N, C;
	cin >> N >> C; // C è il carburante iniziale
	
	BinTree albero;
	
	for (int i=0; i<N; ++i){
		int valore, W;
		cin >> valore >> W;
		albero.inserisci(valore, W);
	}
	
	albero.soluzione(C);
	
	return 0;
}
```

``` cpp
struct Nodo { //2024-09-10
    int valore;
    int h;		//altezza
    
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v, int h) : valore(v), h(h), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;
    vector<pair<int, int>>risultati; // (livello, etichetta)

    void inserisciNodo(Nodo*& u, int v, int h) {
        if (u == nullptr) { 
            u = new Nodo(v, h);
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, h+1);
        else                 
        	inserisciNodo(u->destro,  v, h+1);
    }

    int calcola(Nodo* u) { //postOrder
        if(u == nullptr)
        	return 0;
        	
        if(u->sinistro == nullptr && u->destro == nullptr)
        	return 1;
        	
        int nSx = calcola(u->sinistro);
        int nDx = calcola(u->destro);
        
        int num_foglie = nSx + nDx;
        
        if(num_foglie % 2 == 0)
        	risultati.push_back({u->h, u->valore});
        
        return num_foglie;
    }

public:
    BinTree() {
    	radice = nullptr;
    }

    void inserisci(int v) { 
    	inserisciNodo(radice, v, 0); 
    	}

    void soluzione(int k) { 
        calcola(radice);
        
        sort(risultati.begin(), risultati.end(), 
        	[](auto a, auto b){
        	if(a.first != b.first) return a.first>b.first;
        	return a.second > b.second;
        	}
        );
        
        for(int i=0; i < k && i < risultati.size(); i++){
        	cout << risultati[i].second << endl;
        }
    }     
};

int main() {
	int N, K;
	cin >> N >> K;
	
	BinTree albero;
	
	for (int i=0; i<N; ++i){
		int valore;
		cin >> valore;
		albero.inserisci(valore);
	}
	
	albero.soluzione(K);
	
	return 0;
}
```

``` cpp
struct Nodo { //2025-01-08
    int valore;		// ID
    string nome;
    string stato;
    int costo;		// cost necessario per raggiungerla partendo dal nodo padre
    
    int costo_da_radice;
    
    Nodo* sinistro;
    Nodo* destro;

    Nodo(int v, string nome, string stato, int costo, int r) : valore(v), nome(nome), stato(stato), costo(costo), costo_da_radice(r), sinistro(nullptr), destro(nullptr) {}
};

class BinTree {
    Nodo* radice;
    vector<string>città;
    
    void inserisciNodo(Nodo*& u, int v, string nome, string stato, int costo, int r) {
        if (u == nullptr) { 
            u = new Nodo(v, nome, stato, costo, r);
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, nome, stato, costo, r+u->costo);
        else                 
        	inserisciNodo(u->destro,  v, nome, stato, costo, r+u->costo);
    }

    void calcola(Nodo* u, int M, string C, vector<string> prova) { 
    	if (u == nullptr)
    		return;

	prova.push_back(u->stato);
    		
    	if(u->nome == C){
    		if(u->costo_da_radice > M){
    			return;
    		}
    		città = prova;
    		
    	}
    	    		
    	calcola(u->sinistro, M , C, prova);
    	calcola(u->destro, M, C, prova);
    	
    	// se avessi passato prova per riferimento avrei potuto usare prova.pop_back(); che rimuve lo stato quando torna indietro
        
    }

public:
    BinTree() {
    	radice = nullptr;
    }

    void inserisci(int v, string nome, string stato, int costo) {
    	inserisciNodo(radice, v, nome, stato, costo, costo); 
    	}

    void soluzione(int M, string C) {
    	vector<string> prova; 
        calcola(radice, M, C, prova);
        
        sort(città.begin(), città.end() );
        
        if (città.size() > 0){
		for(int i=0; i < città.size(); i++){
			if(i == 0 || città[i] != città[i-1])
				cout << città[i] << endl;
		}
        }
        else 
        	cout << "NO" << endl;
    }    
       
};

int main() {
	int N, M;
	string C;
	cin >> N >> M >> C;
	
	BinTree albero;
	
	for (int i=0; i<N; ++i){
		int valore, costo;
		string nome, stato;
		cin >> valore >> nome >> stato >> costo;
		albero.inserisci(valore, nome, stato, costo);
	}
	
	albero.soluzione(M, C);
	
	return 0;
}
```

# Tabelle Hash

``` cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

struct Nodo { // 2016-01-27
	int ID;
	string cognome;
	
	Nodo* next;
	
	Nodo(int id, const string& cogn): ID(id), cognome(cogn), next(NULL) {}
};

class HashTable { //Hashtable=banca
	Nodo** table_; 		//tabella hash (array di liste)
	int size_;		//dimensione della tabella (2N)
	
	vector<int> collisioni;	//tiene traccia delle lunghezze delle liste
	int maxCollisioni;	//lunghezza della lista più lunga
	int indiceMaxCollisioni;//indice della lista più lunga
	
	// funzione hash
	int hashFunction(int ID){
		const int p = 999149;
        	const int a = 1000;
        	const int b = 2000;
        	return ((a * ID + b) % p) % size_;
	}
	
public:
	// costruttore
	HashTable (int N){
		size_ = 2*N;
		maxCollisioni = 0;
		indiceMaxCollisioni = 0;
		
		table_ = new Nodo*[size_]();	//inizializza a nullptr
		collisioni.resize(size_,0);	//resize --> funzione di vector --> riempie il vector di 0
	}
	
	
	void inserisci(int id, string& cognome) {
		int indice = hashFunction(id);
		Nodo* nuovo = new Nodo(id, cognome);
		
		//inserimento in testa alla lista
		nuovo->next = table_[indice];
		table_[indice] = nuovo;
		
		//aggiorna collisioni e maxCollisioni
		collisioni[indice]++;
        	if (collisioni[indice] > maxCollisioni || 
        			(collisioni[indice] == maxCollisioni && indice < indiceMaxCollisioni)) { 
        			// OR nell'if controlla il fatto che a parità si scelga quello con indice minore
        	    maxCollisioni = collisioni[indice];
        	    indiceMaxCollisioni = indice;
        	}
	}
	
	//Trova e stampa l'ID del primo elemento nella lista più lunga (ordinato per cognome e ID)
    	void stampaIDPrimoElementoListaLunga() {
	        Nodo* lista = table_[indiceMaxCollisioni];
        	vector<Nodo*> elementi;  // Copiamo gli elementi in un vettore per ordinarli
	
	        // Copia la lista in un vettore
        	Nodo* current = lista;
        	while (current != nullptr) {
        	    elementi.push_back(current);
        	    current = current->next;
        	}

        	// Ordina per cognome (lessicografico) e ID (crescente)
        	sort(elementi.begin(), elementi.end(), 
        	[](const Nodo* a, const Nodo* b) {
		    	if (a->cognome != b->cognome)
		        	return a->cognome < b->cognome;
		    	else
		        	return a->ID < b->ID;
		    	}
            	);
            	
            	// Stampa l'ID del primo elemento
	        if (!elementi.empty()) {
        	    cout << elementi[0]->ID << endl;
        	}
            
             /* 
             la copia della lista nel vettore e il SORT appena fatti non hanno la miglior complessità --> vedi sotto una soluzione con complessità minore
             
             	Nodo* cur  = table_[indiceMaxCollisioni];  // testa della lista //curr scorre la lista
		Nodo* best = cur;                          // candidato minimo iniziale // best è il nodo candidato alla stampa --> avrei potuto salvare int id_da_stampare e string cognome_per_stampa e fare i confronti con quelli
	
		cur = cur->next;                           // partiamo dal secondo nodo
		while (cur) {
		    // se cur è "più piccolo" di best → diventa il nuovo best
		    if (cur->cognome < best->cognome ||
		       (cur->cognome == best->cognome && cur->ID < best->ID))
		        best = cur;
		    cur = cur->next;                       // avanti al nodo successivo
		}
		
		cout << best->ID << endl;                  // ID del nodo migliore
             
             */
            
            
            	
    	}
};

int main(){
	
	//inserimetno di N
	int N;
	cin >> N;
	
	// creazione banca
	HashTable banca(N);		// cra tabella hash di dimensione 2N
	
	//inserimento conti bancari
	for(int i=0; i < N; ++i){
		int id;
		string cognome;
		cin >> id >> cognome;
		banca.inserisci(id, cognome);
	}
	
	// stampa l'ID richiesto
	banca.stampaIDPrimoElementoListaLunga();
	
	return 0;
}
```

tabella hash con liste di trabocco
``` cpp
// 2018-07-19
const int p = 999149;
const int a = 1000;
const int b = 2000;

struct Nodo {
	int ID;
	string cognome;
	
	Nodo* next;
	
	Nodo(int id, string c): ID(id), cognome(c), next(NULL) {}
};

class HashTable { //Hashtable
	Nodo** table; 		//tabella hash (array di liste)
	int size;		//dimensione della tabella
	
	vector<pair<int, int>> collisioni; //(numero collisioni, indice)
	
	
	// funzione hash
	int hashFunction(int ID){
        	return ((a * ID + b) % p) % size;
	}
	
	int trova_minimo(Nodo* testa) {
		if (testa == nullptr) 
			return -1;
			
		string best_cognome = testa->cognome;
		int best_id = testa->ID;
		
		for (Nodo* cur = testa->next; cur; cur = cur->next) {
		    if (cur->cognome < best_cognome ||
		       (cur->cognome == best_cognome && cur->ID < best_id)) {
		        best_cognome = cur->cognome;
		        best_id = cur->ID;
		    }
		}
		return best_id;
	}
	
public:
	// costruttore
	HashTable (int N){
		size = 2*N;
		
		table = new Nodo*[size]();	//inizializza a nullptr
		collisioni.resize(size, {0, 0});//resize --> funzione di vector --> riempie il vector di 0
		for (int i =0; i < size; i++)
			collisioni[i].second = i; //inizializzo indici
	}
	
	void inserisci(int id, string c) {
		int indice = hashFunction(id);
		Nodo* nuovo = new Nodo(id, c);
		
		//inserimento in testa alla lista
		nuovo->next = table[indice];
		table[indice] = nuovo;
		
		//aggiorna collisioni nei vector
		collisioni[indice].first++;
	}
	
    	void soluzione(int k) {
	    	sort(collisioni.begin(), collisioni.end(),
		     	[](auto a, auto b) {
		         if (a.first != b.first) return a.first > b.first;
		         return a.second < b.second;
		     });

		for (int i=0; i<k && i < collisioni.size(); i++) {
		    int indice = collisioni[i].second;
		    int minimo = trova_minimo(table[indice]); //richiamo alla funzione scritta sopra
		    cout << minimo << "\n";
		}
	    }

};

int main(){
	
	//inserimetno di N
	int N, K;
	cin >> N >> K;
	
	// creazione banca
	HashTable tabella_hash(N); //dimensione 2N inizializzata dopo
	
	for(int i=0; i < N; ++i){
		int id;
		string c;
		cin >> id >> c;
		tabella_hash.inserisci(id, c);
	}
	
	tabella_hash.soluzione(K);
	
	return 0;
}
```

``` cpp
// 2019-01-10
// metodo di concatenazione --> liste di trabocco o vector

const int p = 999149;
const int a = 1000;
const int b = 2000;

struct Nodo {
	int ID;
	int età;
	
	Nodo* next;
	
	Nodo(int id, int e): ID(id), età(e), next(NULL) {}
};

class HashTable { //Hashtable
	Nodo** table; 		//tabella hash (array di liste)
	int size;		//dimensione della tabella
	
	vector<pair<int, int>> collisioni; //(numero collisioni, indice)
	
	
	// funzione hash
	int hashFunction(int ID){
        	return ((a * ID + b) % p) % size;
	}
	
	void trova_anziano(int k, int& best_età, int& best_id) {
		for(int i=0; i < collisioni.size() && i < k; i++){
			int indice = collisioni[i].second;
			Nodo* testa = table[indice];
			for (Nodo* cur = testa; cur != nullptr; cur = cur->next) {
			    if(cur->età == best_età)
			    	best_id = min(best_id, cur->ID);
			    else if (cur->età > best_età){
			    	best_età = cur->età;
			    	best_id = cur->ID;
			    	}
			}
		}
		return;
	}
	
public:
	// costruttore
	HashTable (int S){
		size = S;
		
		table = new Nodo*[size]();	//inizializza a nullptr
		collisioni.resize(size, {0, 0});//resize --> funzione di vector --> riempie il vector di 0
		for (int i =0; i < size; i++)
			collisioni[i].second = i; //inizializzo indici
	}
	
	void inserisci(int id, int e) {
		int indice = hashFunction(id);
		Nodo* nuovo = new Nodo(id, e);
		
		//inserimento in testa alla lista
		nuovo->next = table[indice];
		table[indice] = nuovo;
		
		//aggiorna collisioni nei vector
		collisioni[indice].first++;
	}
	
    	void soluzione(int k) {
	    	sort(collisioni.begin(), collisioni.end(),
		     	[](auto a, auto b) {
		         if (a.first != b.first) return a.first > b.first;
		         return a.second < b.second;
		     });

		int best_età = 0;
		int best_id = 0;
		trova_anziano(k, best_età, best_id);
		
		cout << best_id << endl;
		cout << best_età << endl;
	}

};

int main(){
	
	//inserimetno di N
	int N, K, S;
	cin >> N >> K >> S;
	
	// creazione banca
	HashTable tabella_hash(S); 
	
	for(int i=0; i < N; ++i){
		int id;
		int e;
		cin >> id >> e;
		tabella_hash.inserisci(id, e);
	}
	
	tabella_hash.soluzione(K);
	
	return 0;
}

/*
alternativa con ricorsione

void visita(Nodo* cur, int& best_id, int& best_eta) {
    if (!cur) return;
    if (cur->età > best_eta || (cur->età == best_eta && cur->ID < best_id)) {
        best_eta = cur->età;
        best_id = cur->ID;
    }
    visita(cur->next, best_id, best_eta);
}

*/
```

``` cpp
// 2019-07-18
const int p = 999149;
const int a = 1000;
const int b = 2000;

struct Nodo {
	int targa;
	int categoria;
	
	Nodo* next;
	
	Nodo(int t, int c): targa(t), categoria(c), next(NULL) {}
};

class HashTable { //Hashtable
	Nodo** table; 		//tabella hash (array di liste)
	int size;		//dimensione della tabella
	
	vector<pair<int, int>> bro; //(numero veicoli della categoria max, indice tabella)
	
	
	// funzione hash
	int hashFunction(int targa){
        	return ((a * targa + b) % p) % size;
	}
	
	void calcola(int C) {
		for(int i=0; i < size; i++){ //scorro table
			// alloco vector lungo C-1 per gestire il numero di categorie
			vector<int> num_auto_per_categoria;
			num_auto_per_categoria.resize(C, 0); // --> lo riempio di 0 
						//la dimensione è C --> poi gli indici vanno da 0 a C-1
			
			int max=0;
			int categoria_max; 
			
			Nodo* testa = table[i]; //scorro lista e incremento vector
			for (Nodo* cur = testa; cur != nullptr; cur = cur->next) {
			    num_auto_per_categoria[cur->categoria]++;
			    if(num_auto_per_categoria[cur->categoria] > max){
			    	max = num_auto_per_categoria[cur->categoria];
			    	categoria_max = i;
			    }
			    else if(num_auto_per_categoria[cur->categoria] == max)
			    	categoria_max = min(categoria_max, i);
			    	
			}
			bro[i].first = max;
		}
		return;
	}
	
public:
	// costruttore
	HashTable (int C){
		size = C;
		
		table = new Nodo*[size]();	//inizializza a nullptr
		bro.resize(size, {0, 0});//resize --> funzione di vector --> riempie il vector di 0
		for (int i =0; i < size; i++){
			bro[i].second = i; //inizializzo indici
		}
	}
	
	void inserisci(int t, int c) {
		int indice = hashFunction(t);
		Nodo* nuovo = new Nodo(t, c);
		
		//inserimento in testa alla lista
		nuovo->next = table[indice];
		table[indice] = nuovo;
		
	}
	
    void soluzione(int k, int C) {
    	
    		calcola(C);

	    	sort(bro.begin(), bro.end(),
		     	[](auto a, auto b) {
		         if (a.first != b.first) return a.first > b.first;
		         return a.second < b.second;
		     });

		for(int i= 0; i<k && i<bro.size(); i++)
			cout << bro[i].second << endl;
	}

};

int main(){
	
	//inserimetno di N
	int N, K, C;
	cin >> N >> K >> C;
	
	// creazione banca
	HashTable tabella_hash(C); 
	
	for(int i=0; i < N; ++i){
		int t, c;
		cin >> t >> c;
		tabella_hash.inserisci(t, c);
	}
	
	tabella_hash.soluzione(K, C);
	
	return 0;
}
```

``` cpp
const int p = 999149; //2025-02-11
const int a = 1000;
const int b = 2000;

struct Nodo {
	int ID;
	string nome;
	float PM; 	//prezzo medio
	int numero_prenotazioni;
	
	Nodo* next;
	
	Nodo(int id, string n): ID(id), nome(n), PM(-1), numero_prenotazioni(0), next(NULL) {}
};

class HashTable { //Hashtable
	Nodo** table; 		//tabella hash (array di liste)
	int size;		//dimensione della tabella

	// funzione hash
	int hashFunction(int x){
        	return ((a * x + b) % p) % size;
	}
	
	
public:
	// costruttore
	HashTable (int N){
		size = N*2;
		table = new Nodo*[size]();	//inizializza a nullptr
		
	}
	
	void inserisciHotel(int id, string n) {
		int indice = hashFunction(id);
		Nodo* nuovo = new Nodo(id, n);
		
		//inserimento in testa alla lista
		nuovo->next = table[indice];
		table[indice] = nuovo;
		
	}
	
	void inserisciPrenotazione(int H, float P){
		//trovare hotel con id==H
		int indice = hashFunction(H);
		for(Nodo* cur = table[indice]; cur != nullptr; cur = cur->next){
			if(cur->ID == H){ //calcolo PM
				if(cur->PM == -1){
					cur->PM = P;
					cur->numero_prenotazioni++;
					return;
				}
				else {
					cur->PM = (cur->PM * cur->numero_prenotazioni + P)/ (cur->numero_prenotazioni +1 );
					cur->numero_prenotazioni++;					
					return;
				}
			}
		}
		

	}
	
    void soluzione(int k) {
    		vector<pair<float, string>> hotel; //(PM, nome_hotel)
    
    		//raccolgo tutti gli hotel con alemeno 1 prenotazione
    		for(int i = 0; i < size; i++){
		    for(Nodo* cur = table[i]; cur != nullptr; cur = cur->next){
		        if(cur->numero_prenotazioni > 0){
		            hotel.push_back({cur->PM, cur->nome});
		        }
		    }
		}

	    	sort(hotel.begin(), hotel.end(),
		     	[](auto a, auto b) {
		         if (a.first != b.first) return a.first < b.first; //PM cescente
		         return a.second < b.second; //ordine lessicografico per nome
		     });

		for(int i= 0; i<k && i<hotel.size(); i++)
			cout << hotel[i].second << endl;
	}
};

int main(){
	//inserimetno di N
	int N, M, K;
	cin >> N >> M >> K;
	
	// creazione banca
	HashTable tabella_hash(N); 
	
	//inserimento hotel
	for(int i=0; i < N; ++i){
		int id;
		string n;
		cin >> id >> n;
		tabella_hash.inserisciHotel(id, n);
	}
	
	//inserisci prenotazione
	for(int i= 0; i < M; i++){
		int H;
		float P;
		cin >> H >> P;
		tabella_hash.inserisciPrenotazione(H, P);
	}
	
	tabella_hash.soluzione(K);
	
	return 0;
}
```

# Tabella Hash con ABR
tabella hash fatta con vector e non con array
``` cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

const int a = 1000; // 2016-09-12
const int b = 2000;
const int p = 999149;

// Nodo dell'albero
struct NodoABR {
    int valore;
    
    NodoABR* sinistro;
    NodoABR* destro;
    
    // Costruttore
    NodoABR(int val) : valore(val), sinistro(nullptr), destro(nullptr) {}
};

// Classe per l'albero binario di ricerca
class ABR{
    NodoABR* radice;
    
    int altezza_albero;

    void inserisciNodo(NodoABR*& u, int v, int h){
    	if (u == nullptr) {
            u = new NodoABR(v);
            
            if (h > altezza_albero)
            	altezza_albero = h;
            return;
        }
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, h+1);
        else                 
        	inserisciNodo(u->destro, v, h+1);
    }


public:
    ABR(){
    	radice=nullptr;
    	altezza_albero = -1; //alberi vuoti hanno altezza -1
    }

    void inserisci(int val) {
        inserisciNodo(radice, val, 0);
    }
    
    int getAltezza() {
    	return altezza_albero;
    }

   
};

//stuttura e funzione per risoluzione esercizio

// Struttura per rappresentare indirizzo-altezza per l'ordinamento
struct IndirizzoAltezza {
    int indirizzo;
    int altezza;
    
    IndirizzoAltezza(int ind, int alt) : indirizzo(ind), altezza(alt) {}
};

// Funzione di confronto per l'ordinamento
bool confronta(const IndirizzoAltezza& a, const IndirizzoAltezza& b) { //avrei potuto metterla nel sort
    if (a.altezza != b.altezza) {
        return a.altezza > b.altezza;  // Altezza decrescente
    }
    return a.indirizzo < b.indirizzo;  // A parità di altezza, indirizzo crescente
}


// Classe per la tabella hash
class HashTable {
    vector <ABR> tabella; //vettore di ABR
    int dimensione; //dimensione dela tabella (S)
    
    // Funzione hash
	int hashFunction(int x, int S) {
	    return ((a * x + b) % p) % S;
	}

public:
    //costruttore
    HashTable( int S){
    	dimensione = S;
    	
    	tabella.resize(S); //riempie il vecot di 0
    }
    
    void inserisciValore(int x){
    	int indice = hashFunction(x, dimensione);
    	
    	tabella[indice].inserisci(x); //questa 'inserisci' fa riferimento alla funzione public della classe ABR 
    }
    
    //funzione per ottenere i primi K indirizzi ordinati
    void stampaKindirizzi(int K) {
    vector <IndirizzoAltezza> indirizzi_altezze;
    
    // Raccogli tutti gli indirizzi con altezza > -1 (non vuoti)
        for (int i = 0; i < dimensione; i++) {
            int altezza = tabella[i].getAltezza();
            if (altezza > -1) {  // Solo ABR non vuoti
                indirizzi_altezze.push_back(IndirizzoAltezza(i, altezza)); //se l'altezza è valida inserisce indirizzo e altezza in fondo al vector di IndirizzoALtezza
            }
        }
        
        // Ordina secondo i criteri richiesti
        sort(indirizzi_altezze.begin(), indirizzi_altezze.end(), confronta);
        
        // Stampa i primi K indirizzi (o tutti se sono meno di K)
        int num_da_stampare = min(K, (int)indirizzi_altezze.size()); //il minimo tra K e la dimensione del vector (nel caso in cui fosse minore di k) //è necessario il cast esplicito (int) perchè sennò paragoniamo due tipi diversi --> int Vs size
        /*
        alternativa
        
        int size_vector = inidrizzi_altezze.size();
        int num_da_stampare = min(k, size_vector);
        */
        for (int i = 0; i < num_da_stampare; i++) {
            cout << indirizzi_altezze[i].indirizzo << endl; //stampa solo gli indirizzi e non le altezze
        }
    }
};

int main(){

	// inserimento di N, K e S
	int N, K, S;
	cin >> N >> K >> S;
	
	//creazione tabella Hash
	HashTable tabella_hash(S);
	
	
	//inserimento N nodi
	for (int i=0; i<N; ++i){
	int x;
	cin >> x;
	tabella_hash.inserisciValore(x);
	}
	
	
	
	//stampa dei primi k indirizzi
	tabella_hash.stampaKindirizzi(K);
	
	return 0;
}


```

``` cpp
const int a = 1000; // 2017-06-29
const int b = 2000;
const int p = 999149;

// Nodo dell'albero
struct NodoABR {
    int valore;
    int num_duplicati;
    
    NodoABR* sinistro;
    NodoABR* destro;
    
    NodoABR(int val) : valore(val), num_duplicati(1), sinistro(nullptr), destro(nullptr) {}
};

// Classe per l'albero binario di ricerca
class ABR{
    NodoABR* radice;
    int etichetta_D;
    int num_max_duplicati;

    void inserisciNodo(NodoABR*& u, int v){
    	if (u == nullptr) {
            u = new NodoABR(v); // quando creo il nodo il valore di num_duplicati viene impostato a 1
            if (u->num_duplicati > num_max_duplicati){
            	etichetta_D = u->valore;
        		num_max_duplicati = u->num_duplicati;
            }
            
            return;
        }
        if (v == u->valore){
        	u->num_duplicati++;
        	if (u->num_duplicati > num_max_duplicati){
        		etichetta_D = u->valore;
        		num_max_duplicati = u->num_duplicati;
        		}
        	else if (u->num_duplicati == num_max_duplicati)
        		etichetta_D = (etichetta_D > u->valore ? etichetta_D : u->valore); // etichetta_D = max(etichetta_D, u->valore)
        }
        if (v < u->valore) 
        	inserisciNodo(u->sinistro, v);
        else                 
        	inserisciNodo(u->destro, v);
    }


public:
    ABR(){
    	radice=nullptr;
    	num_max_duplicati = 0;
    	etichetta_D = -1;
    }

    void inserisci(int val) {
        inserisciNodo(radice, val);
    }
    
    int getD(){
    	return etichetta_D;
    }
   
};

// Classe per la tabella hash
class HashTable {
    vector <ABR> tabella; //vettore di ABR
    int dimensione; //dimensione dela tabella (S)
    
    // Funzione hash
	int hashFunction(int x, int S) {
	    return ((a * x + b) % p) % S;
	}

public:
    //costruttore
    HashTable( int S){
    	dimensione = S;
    	
    	tabella.resize(S); //riempie il vecot di 0
    }
    
    void inserisciValore(int x){
    	int indice = hashFunction(x, dimensione);
    	
    	tabella[indice].inserisci(x); //questa 'inserisci' fa riferimento alla funzione public della classe ABR 
    }
    
    void stampa(int k){
    	// creo un  vector con tutti i valori D e poi li ordino 
    	vector<int> valori;
    	for(int i = 0; i < dimensione; i++){
    		int d = tabella[i].getD();
    		if (d >= 0)
    			valori.push_back(d);
    	}
    	
    	sort(valori.begin(), valori.end(), 
    		[](int x, int y){
    			return x > y; //ordine decrescente
    		}
    		);
    	for(int i=0; i < k && i< (int)valori.size(); ++i){
    		cout << valori[i] << endl;
    	}
    	
    }
    
};

int main(){

	// inserimento di N, K e S
	int N, K, S;
	cin >> N >> K >> S;
	
	//creazione tabella Hash
	HashTable tabella_hash(S);
	
	
	//inserimento N nodi
	for (int i=0; i<N; ++i){
	int x;
	cin >> x;
	tabella_hash.inserisciValore(x);
	}
	
	//stampa dei primi k indirizzi
	tabella_hash.stampa(K);
	
	return 0;
}
```

``` cpp
const int a = 1000; // 2018-01-11
const int b = 2000;
const int p = 999149;

// Nodo dell'albero
struct NodoABR {
    int valore;
    
    NodoABR* sinistro;
    NodoABR* destro;
    
    NodoABR(int val) : valore(val), sinistro(nullptr), destro(nullptr) {}
};

// Classe per l'albero binario di ricerca
class ABR{
    NodoABR* radice;
    int ndx;
    int nsx;

    void inserisciNodo(NodoABR*& u, int v){
    	if (u == nullptr) {
            u = new NodoABR(v); 
                        
            return;
        }
        
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v);
        else                 
        	inserisciNodo(u->destro, v);
    }
    
    void calcolaPrivato(NodoABR*& u){
    	if(u == nullptr)
    		return;
    		
    	if(u->sinistro != nullptr && //guardo se mio figlio sinistro è un candidato
    		u->sinistro->sinistro == nullptr && u->sinistro->destro == nullptr)
    		nsx++;
    	
    	if(u->destro != nullptr && // guardo se mio figlio destro è un candidato
    		u->destro->sinistro == nullptr && u->destro->destro==nullptr)
    		ndx++;
    	
    	calcolaPrivato(u->destro);
    	calcolaPrivato(u->sinistro);
    }


public:
    ABR(){
    	radice=nullptr;
    	ndx = 0;
    	nsx = 0;
    }

    void inserisci(int val) {
        inserisciNodo(radice, val);
    }
    
    int getNdx(){
    	return ndx;
    }
    
    int getNsx(){
    	return nsx;
    }
    
    void calcola(){
    	calcolaPrivato(radice);
    }
   
};


// Classe per la tabella hash
class HashTable {
	vector <ABR> tabella; //vettore di ABR
	int dimensione; //dimensione dela tabella (S)
	    
	int max_ndx;
	int max_nsx;
	
	int indirizzo_ndx;
	int indirizzo_nsx;

	// Funzione hash
	int hashFunction(int x, int S) {
		//int indirizzo = ((a * x + b) % p) % S;
		//cout << "elemento " << x << " inserito all'indirizzo " << indirizzo << endl;
		//return indirizzo;
	    return ((a * x + b) % p) % S;
	}
	
	void stampa(){
		for(int i=0; i < dimensione; i++){
			int right = tabella[i].getNdx();
			if (right == max_ndx){
				indirizzo_ndx = max(i, indirizzo_ndx);
			}
			else if (right > max_ndx){
				max_ndx = right;
				indirizzo_ndx = i;
			}
			
			int left = tabella[i].getNsx();
			if (left == max_nsx){
				indirizzo_nsx = max(i, indirizzo_nsx);
			}
			else if (left > max_nsx){
				max_nsx = left;
				indirizzo_nsx = i;
			}
		}
		
		cout << indirizzo_ndx << endl;
		cout << indirizzo_nsx << endl;
	}

public:
    //costruttore
    HashTable( int S){
    	dimensione = S;
    	max_ndx=0;
    	max_nsx=0;
    	indirizzo_ndx = 0;
    	indirizzo_nsx = 0;
    	
    	tabella.resize(S); //riempie il vecot di 0
    }
    
    void inserisciValore(int x){
    	int indice = hashFunction(x, dimensione);
    	
    	tabella[indice].inserisci(x); //questa 'inserisci' fa riferimento alla funzione public della classe ABR 
    }
    
    void soluzione(){
    	for(int i=0; i < dimensione; ++i){
    		tabella[i].calcola();
    	}
    	
    	stampa();
    }
};

int main(){

	int N, S;
	cin >> N >> S;
	
	//creazione tabella Hash
	HashTable tabella_hash(S);
	
	//inserimento N nodi
	for (int i=0; i<N; ++i){
	int x;
	cin >> x;
	tabella_hash.inserisciValore(x);
	}
	
	tabella_hash.soluzione();
	
	return 0;
}

/*
si poteva evitare la calcoloPrivato, calcolando ndx e nsx durante l'inserimento

void inserisciNodo(NodoABR*& u, int v, NodoABR* padre) {
        if (u == nullptr) {
            u = new NodoABR(v, padre);

            // nuovo nodo è foglia
            if (padre != nullptr) {
                if (padre->sinistro == u) {
                    nsx++;
                    // se il padre era foglia sinistra → non più foglia
                    if (padre->sinistro == u && padre->destro == nullptr && padre->padre != nullptr) {
                        if (padre->padre->sinistro == padre) nsx--;
                        else ndx--;
                    }
                } else {
                    ndx++;
                    // se il padre era foglia destra → non più foglia
                    if (padre->destro == u && padre->sinistro == nullptr && padre->padre != nullptr) {
                        if (padre->padre->sinistro == padre) nsx--;
                        else ndx--;
                    }
                }
            }
            return;
        }

*/
```

``` cpp
const int a = 1000; // 2024-06-25
const int b = 2000;
const int p = 999149;

// Nodo dell'albero
struct NodoABR {
    int valore;
    string stringa;
    
    int MaxSx;		// lunghezza massima tra tutte le stringhe del sottoalbero sinistro di x
    int MaxDx;		// lunghezza massima tra tutte le stringhe del sottoalbero destro di x
    int lev;		// altezza
    
    NodoABR* sinistro;
    NodoABR* destro;
    
    NodoABR(int val, string stringa, int h) : valore(val), stringa(stringa), MaxDx(0), MaxSx(0), lev(h), sinistro(nullptr), destro(nullptr) {}
};

// Classe per l'albero binario di ricerca
class ABR{
    NodoABR* radice;
    int V; //num nodi validi per questo albero

    void inserisciNodo(NodoABR*& u, int v, string stringa, int h){
    	if (u == nullptr) {
            u = new NodoABR(v, stringa, h); 
                        
            return;
        }
        
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, stringa, h+1);
        else                 
        	inserisciNodo(u->destro, v, stringa, h+1);
    }
    
    int calcolaPrivato(NodoABR*& u){
    	if(u == nullptr)
    		return 0;
    	if(u->sinistro == nullptr && u->destro == nullptr){
    		if(abs(u->MaxDx - u->MaxSx) <= u->lev)
    			V++;
    		return u->stringa.size();
    	}
    	
    	
    	int max_size_dx = calcolaPrivato(u->destro);
    	int max_size_sx = calcolaPrivato(u->sinistro);
    		
    	u->MaxDx = max_size_dx;
    	u->MaxSx = max_size_sx;
    	
    	if(abs(u->MaxDx - u->MaxSx) <= u->lev)
    		V++;
    		
    	int max_size= max(max_size_dx, max_size_sx);
    	
    	return max(max_size, (int)u->stringa.size());
    		
    }

public:
    ABR(){
    	radice=nullptr;
    	V = 0;
    }

    void inserisci(int val, string stringa) {
        inserisciNodo(radice, val, stringa, 0);
    }
    
    void calcola(){
    	calcolaPrivato(radice);
    }
    
    int getV (){
    	return V;
    }
};


// Classe per la tabella hash
class HashTable {
	vector<ABR> tabella; //vettore di ABR
	int dimensione; //dimensione dela tabella (S)

	// Funzione hash
	int hashFunction(int x, int S) {
	    return ((a * x + b) % p) % S;
	}
	
	void stampa(int k){
		for(int i=0; i < dimensione; i++){
			int amo = tabella[i].getV();
			if (amo >= k)
				cout << i << endl;
		}
	}

public:
    //costruttore
    HashTable( int M){
    	dimensione = M;
    	
    	tabella.resize(M); //riempie il vector di 0
    }
    
    void inserisciValore(int x, string stringa){
    	int indice = hashFunction(x, dimensione);
    	
    	tabella[indice].inserisci(x, stringa); //questa 'inserisci' fa riferimento alla funzione public della classe ABR 
    }
    
    void soluzione(int k){
    	for(int i=0; i < dimensione; ++i)
    		tabella[i].calcola();
    	
    	stampa(k);
    }

};

int main(){

	int N, K, M;
	cin >> N >> K >> M;
	
	//creazione tabella Hash
	HashTable tabella_hash(M);
	
	//inserimento N nodi
	for (int i=0; i<N; ++i){
		int x;
		string stringa;
		cin >> x >> stringa;
		tabella_hash.inserisciValore(x, stringa);
	}
	
	tabella_hash.soluzione(K);
	
	return 0;
}
```

``` cpp
const int a = 1000; // 2025-06-09
const int b = 2000;
const int p = 999149;

struct NodoABR {
    int matricola;
    int categoria;
    float spesa;
    
    NodoABR* sinistro;
    NodoABR* destro;
    
    NodoABR(int mat, int cat, float sp) 
        : matricola(mat), categoria(cat), spesa(sp), sinistro(nullptr), destro(nullptr) {}
};

class ABR {
private:
    NodoABR* radice;
    
    int C; // numero categorie
    vector<float> spesa_totale;   //somma totale delle spese --> per categoria (dato dall'indice)
    vector<int> num_missioni;   //numero di misioni --> per categoria (dato dall'indicce)
    
    void inserisciNodo(NodoABR*& u, int matricola, int categoria, float spesa) {
        if (u == nullptr) {
            u = new NodoABR(matricola, categoria, spesa);
            
            // Aggiorna statistiche per questa categoria
            spesa_totale[categoria] += spesa;
            num_missioni[categoria]++;
            
            return;
        }
        
        if (matricola <= u->matricola) {
            inserisciNodo(u->sinistro, matricola, categoria, spesa);
        } else {
            inserisciNodo(u->destro, matricola, categoria, spesa);
        }
    }
    
    void inorderStampa(NodoABR* u, int categoriaTarget){
        if (u == nullptr) 
        	return;
        	
        inorderStampa(u->sinistro, categoriaTarget);
        if (u->categoria == categoriaTarget)
            cout << u->matricola << " ";
        inorderStampa(u->destro, categoriaTarget);
    }

public:
    ABR(int num_categorie){
    	radice = nullptr;
    	C = num_categorie;
        spesa_totale.resize(C, 0.0);
        num_missioni.resize(C, 0);
    }
    
    void inserisci(int matricola, int categoria, float spesa) {
        inserisciNodo(radice, matricola, categoria, spesa);
    }
    
    void stampaCategoriaMax() {
        int categoria_max = -1;
        float spesa_media_max = -1.0;
        
        for (int i = 0; i < C; i++) {
            if (num_missioni[i] > 0) {
                float spesa_media = spesa_totale[i] / num_missioni[i];
                
                // A parità di spesa media, scegli categoria con valore minore
                if (spesa_media > spesa_media_max || 
                    (spesa_media == spesa_media_max && (categoria_max == -1 || i < categoria_max))) {
                    spesa_media_max = spesa_media;
                    categoria_max = i;
                }
            }
        }
        
        if (categoria_max == -1) {
            cout << "\n"; // bucket vuoto
            return;
        }

        // Stampa matricole in ordine
        inorderStampa(radice, categoria_max);
        cout << endl;
    }

};

class HashTable {
    vector<ABR> tabella;
    int dimensione;
    
    int hashFunction(int x) {
        return ((a * x + b) % p) % dimensione;
    }
    
public:
    HashTable(int C) {
    	dimensione = C;
    	
        for (int i = 0; i < C; i++) {
            tabella.push_back(ABR(C));
        }
    }
    
    void inserisci(int matricola, int categoria, float spesa) {
        int indice = hashFunction(matricola);
        tabella[indice].inserisci(matricola, categoria, spesa);
    }
    
    void soluzione(){
    	for(int i=0; i < dimensione; ++i)
    		tabella[i].stampaCategoriaMax();
    	
    }
};

int main() {
    int N, C;
    cin >> N >> C;
    
    HashTable tabella(C);
    
    for (int i = 0; i < N; i++) {
        int matricola, categoria;
        float spesa;
        cin >> matricola >> categoria >> spesa;
        tabella.inserisci(matricola, categoria, spesa);
    }
    
    tabella.soluzione();
    
    return 0;
}
```

``` cpp
const int a = 1000; //2025-07-18
const int b = 2000;
const int p = 999149;

// Nodo dell'albero
struct NodoABR {
    int valore;		//id
    string categoria;
    
    NodoABR* sinistro;
    NodoABR* destro;
    
    NodoABR(int val, string stringa) : valore(val), categoria(stringa), sinistro(nullptr), destro(nullptr) {}
};

// Classe per l'albero binario di ricerca
class ABR{
    NodoABR* radice;
    int num_foglie_della_categoria;

    void inserisciNodo(NodoABR*& u, int v, string stringa){
    	if (u == nullptr) {
            u = new NodoABR(v, stringa); 
                        
            return;
        }
        
        if (v <= u->valore) 
        	inserisciNodo(u->sinistro, v, stringa);
        else                 
        	inserisciNodo(u->destro, v, stringa);
    }
    
    void calcolaPrivato(NodoABR*& u, string C){
    	if(u == nullptr)
    		return;
    	
    	if(u->sinistro == nullptr && u->destro == nullptr && u->categoria == C)
    		num_foglie_della_categoria++;
    	
    	calcolaPrivato(u->sinistro, C);
    	calcolaPrivato(u->destro, C);
    	
    }

public:
    ABR(){
    	radice=nullptr;
    	num_foglie_della_categoria = 0;
    }

    void inserisci(int val, string stringa) {
        inserisciNodo(radice, val, stringa);
    }
    
    void calcola(string C){
    	calcolaPrivato(radice, C);
    }
    
    int getNumFoglieCategoria(){
    	return num_foglie_della_categoria;
    }
};


// Classe per la tabella hash
class HashTable {
	vector<ABR> tabella; //vettore di ABR
	int dimensione; //dimensione dela tabella (S)
	
	int indice_categoria_max;

	// Funzione hash
	int hashFunction(int x) {
	    return ((a * x + b) % p) % dimensione;
	}
	

public:
    //costruttore
    HashTable( int N){
    	dimensione = N;
    	indice_categoria_max = -1;
    	
    	tabella.resize(N); //riempie il vector di 0
    }
    
    void inserisciValore(int x, string stringa){
    	int indice = hashFunction(x);
    	
    	tabella[indice].inserisci(x, stringa); //questa 'inserisci' fa riferimento alla funzione public della classe ABR 
    }
    
    void soluzione(string C){
    	int best_num_foglie= -1; 
    	for(int i=0; i < dimensione; ++i){
    		tabella[i].calcola(C);

    		if (best_num_foglie == tabella[i].getNumFoglieCategoria() && tabella[i].getNumFoglieCategoria() != 0)
    			indice_categoria_max = max(indice_categoria_max, i);
    		else if (tabella[i].getNumFoglieCategoria() > best_num_foglie && tabella[i].getNumFoglieCategoria() != 0){
    			indice_categoria_max = i;
    			
    			best_num_foglie = tabella[i].getNumFoglieCategoria();
    		}
    		
    		}
    	
    	cout << indice_categoria_max << endl;
    }

};

int main(){

	int N;
	string C;
	cin >> N >> C;
	
	//creazione tabella Hash
	HashTable tabella_hash(N);
	
	//inserimento N nodi
	for (int i=0; i<N; ++i){
		int x;
		string stringa;
		cin >> x >> stringa;
		tabella_hash.inserisciValore(x, stringa);
	}
	
	tabella_hash.soluzione(C);
	
	return 0;
}
```

# Tabella Hash con lista di trabocco + ABR

``` cpp
const int a = 1000; // 2025-01-24
const int b = 2000;
const int p = 999149;

// Nodo dell'ABR dei corsi
struct CorsoNode {
    int codice;
    int iscritti;
    
    CorsoNode* sinistro;
    CorsoNode* destro;

    CorsoNode(int c) : codice(c), iscritti(1), sinistro(nullptr), destro(nullptr) {}
};

// Classe ABR per i corsi di un docente
class ABR {
    CorsoNode* radice;
    
    string max_cognome_albero;
    int max_iscritti_albero;

    void inserisciNodo(CorsoNode*& u, int codice) {
        if (u == nullptr) {
            u = new CorsoNode(codice);
            
            max_iscritti_albero = max(max_iscritti_albero, 1);
            return ;
        }
        
        if (codice == u->codice) {
            u->iscritti++;
            max_iscritti_albero = max(max_iscritti_albero, u->iscritti);
        } 
        else if (codice < u->codice)
            inserisciNodo(u->sinistro, codice);
        else
            inserisciNodo(u->destro, codice);
        
    }

public:
    ABR() {
    	radice = nullptr;
    	max_iscritti_albero = 0;
    }

    void inserisci(int codice) {
        inserisciNodo(radice, codice);
    }
    
    void setCognomeDocente(const string& cognome){
    	max_cognome_albero = cognome;
    }
    
    string getCognome(){
    	return max_cognome_albero;
    }
    
    int getMax_iscritti_albero(){
    	return max_iscritti_albero;
    }
};

// Nodo della lista di trabocco
struct NodoDocente {
    int matricola;
    string cognome;
    ABR corsi; //ABR contenente i corsi del docente
    NodoDocente* next;

    NodoDocente(int m, const string c) : matricola(m), cognome(c), next(nullptr) {corsi.setCognomeDocente(cognome);}
};

// Classe Tabella Hash con liste di trabocco
class TabellaHash {
    vector<NodoDocente*> tabella;
    int dimensione;

    int max_iscritti;
    string max_cognome;

    int funzioneHash(int x) {
        return ((a * x + b) % p) % dimensione;
    }
    

public:
    TabellaHash(int N){
        dimensione = 2*N;
        max_iscritti = -1;
        max_cognome = "";
        
        tabella.resize(dimensione, nullptr);
    }

    void inserisciDocente(int matricola, string cognome) {
        int indice = funzioneHash(matricola);
        NodoDocente* nuovo = new NodoDocente(matricola, cognome);
        
        nuovo->next = tabella[indice]; //inserimento in testa
        tabella[indice] = nuovo;
    }

    void inserisciIscrizione(int matricola, int codiceCorso) {
	int indice = funzioneHash(matricola);
	
	//cerca docente nella lista di trabocco
	for(NodoDocente* cur = tabella[indice]; cur != nullptr; cur = cur->next){
		if(cur->matricola == matricola){
			cur->corsi.inserisci(codiceCorso);
                        
                        int iscritti_albero = cur->corsi.getMax_iscritti_albero();
                        string cognome_albero = cur->corsi.getCognome();
                                  
                        if (iscritti_albero > max_iscritti || 
		            (iscritti_albero == max_iscritti && cognome_albero < max_cognome)) {
		            max_iscritti = iscritti_albero;
		            max_cognome = cognome_albero;
		        }
                                  
			return;
		}
	}

    }

    void stampaSoluzione() {
        cout << max_cognome << endl;
    }
};

int main() {

    int N, M;
    cin >> N >> M;

    // creazione tabella hash
    TabellaHash tabella(N);

    // inserimento docenti
    for (int i = 0; i < N; i++) {
        int matricola;
        string cognome;
        cin >> matricola >> cognome;
        tabella.inserisciDocente(matricola, cognome);
    }

    // inserimento iscrizioni
    for (int i = 0; i < M; i++) {
        int matricola, codiceCorso;
        cin >> matricola >> codiceCorso;
        tabella.inserisciIscrizione(matricola, codiceCorso);
    }

    // stampa soluzione
    tabella.stampaSoluzione();

    return 0;
}
```

# Tabella Hash + heap

``` cpp
#include <iostream>
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>
using namespace std;

// ==========================
// Struttura servizio
// ==========================
struct Servizio { //2025-06-27 versione chatgpt --> non supera 1 testset
    string ID;
    string categoria;
    int gradimento;
    
    Servizio(string id="", string cat="", int g=0) 
        : ID(id), categoria(cat), gradimento(g) {}
};

// ==========================
// Classe Heap
// ==========================
class Heap {
    vector<Servizio> A; //array che contiene i nodi

    // funzione di confronto: restituisce true se a deve stare sopra b
    bool priorita(const Servizio& a, const Servizio& b) {
        if (a.gradimento != b.gradimento) 
            return a.gradimento > b.gradimento; // gradimento maggiore prima
        return a.ID < b.ID; // a parità di gradimento, ID minore prima
    }

    void heapifyUp(int i) { //risale finchè il figlio ha priorità maggiore del padre
        while (i > 0) {
            int p = (i - 1) / 2;
            if (priorita(A[i], A[p])) {
                swap(A[i], A[p]);
                i = p;
            } else break;
        }
    }

    void heapifyDown(int i) { //scende finchè un figlio ha priorità maggiore del nodo corrente
        int n = A.size();
        while (true) {
            int l = 2*i + 1;
            int r = 2*i + 2;
            int best = i;

            if (l < n && priorita(A[l], A[best])) best = l;
            if (r < n && priorita(A[r], A[best])) best = r;

            if (best != i) {
                swap(A[i], A[best]);
                i = best;
            } else break;
        }
    }

public:
    Heap() {}

    void push(const Servizio& s) { //inserimento
        A.push_back(s); //inserimento in fondo
        heapifyUp(A.size()-1); //sistemo l'albero
    }

    bool empty() { return A.empty(); }

    Servizio top() { return A.front(); }

    void pop() {
        if (A.empty()) return;
        
        A[0] = A.back(); //sposto l'ultimo in cima
        A.pop_back();
        if (!A.empty()) heapifyDown(0); //lo faccio scendere finchè serve
    }
};

// ==========================
// Classe HashTable
// ==========================
class HashTable {
    vector<Heap> table;
    int size;

    // funzione hash data dalla traccia
    int hashFunction(const string& x, int N) {
        int h = 0;
        for (char c : x) {
            h += (c % (N/2));
        }
        return h % size;
    }

public:
    HashTable(int N) {
        size = N; 
        table.resize(size);
    }

    void inserisci(const Servizio& s, int N) {
        int idx = hashFunction(s.ID, N);
        table[idx].push(s);
    }

    string soluzione(int K) {
        unordered_map<string,int> freq; // frequenza categorie

        for (int i=0; i<size; i++) {
            Heap temp = table[i]; // copia heap
            for (int k=0; k<K && !temp.empty(); k++) {
                Servizio s = temp.top();
                temp.pop();
                freq[s.categoria]++;
            }
        }

        // Trova la categoria con max frequenza (in ordine alfabetico a parità)
        string bestCat;
        int bestCnt = -1;
        for (auto& p : freq) {
            if (p.second > bestCnt || (p.second == bestCnt && p.first < bestCat)) {
                bestCat = p.first;
                bestCnt = p.second;
            }
        }

        return bestCat;
    }
};

// ==========================
// MAIN
// ==========================
int main() {
    int N, K;
    cin >> N >> K;

    HashTable tabella(N);

    for (int i=0; i<N; i++) {
        string id, cat;
        int g;
        cin >> id >> cat >> g;
        tabella.inserisci(Servizio(id, cat, g), N);
    }

    cout << tabella.soluzione(K) << "\n";

    return 0;
}

```

``` cpp
#include <iostream>
#include <vector>
#include <string>
#include <map>
#include <algorithm>
using namespace std;

struct Servizio { //2025-06-27 versione claude
    string ID;
    string categoria;
    int gradimento;
    
    Servizio(string id, string cat, int grad) : ID(id), categoria(cat), gradimento(grad) {}
};

// Classe Heap riutilizzabile per i servizi
class MaxHeap {
    vector<Servizio> heap;
    
    // Funzione di comparazione: gradimento decrescente, poi ID crescente
    bool compare(const Servizio& a, const Servizio& b) { //implementa logica di ordinamento
        if (a.gradimento != b.gradimento) 
            return a.gradimento > b.gradimento; // gradimento decrescente
        return a.ID < b.ID; // ID crescente
    }
    
    // Sposta verso l'alto (per inserimento)
    void heapifyUp(int index) {
        while (index > 0) {
            int parent = (index - 1) / 2;
            if (compare(heap[index], heap[parent])) {
                swap(heap[index], heap[parent]);
                index = parent;
            } else {
                break;
            }
        }
    }
    
    // Sposta verso il basso (per estrazione)
    void heapifyDown(int index) {
        int size = heap.size();
        while (true) {
            int largest = index;
            int left = 2 * index + 1;
            int right = 2 * index + 2;
            
            if (left < size && compare(heap[left], heap[largest]))
                largest = left;
            
            if (right < size && compare(heap[right], heap[largest]))
                largest = right;
            
            if (largest != index) {
                swap(heap[index], heap[largest]);
                index = largest;
            } else {
                break;
            }
        }
    }

public:
    // Inserisce un servizio nell'heap
    void inserisci(const Servizio& servizio) {
        heap.push_back(servizio);
        heapifyUp(heap.size() - 1);
    }
    
    // Estrae i primi K servizi senza rimuoverli dall'heap
    vector<Servizio> estraiPrimiK(int k) {
        vector<Servizio> risultato;
        vector<Servizio> temp = heap; // copia per non modificare l'originale
        
        int n = min(k, (int)temp.size());
        
        // Estrai i primi k elementi
        for (int i = 0; i < n; i++) {
            if (temp.empty()) break;
            
            risultato.push_back(temp[0]);
            
            // Rimuovi il primo elemento e riorganizza
            temp[0] = temp.back();
            temp.pop_back();
            
            // Riorganizza l'heap temporaneo
            if (!temp.empty()) {
                int index = 0;
                int size = temp.size();
                while (true) {
                    int largest = index;
                    int left = 2 * index + 1;
                    int right = 2 * index + 2;
                    
                    if (left < size && compare(temp[left], temp[largest]))
                        largest = left;
                    
                    if (right < size && compare(temp[right], temp[largest]))
                        largest = right;
                    
                    if (largest != index) {
                        swap(temp[index], temp[largest]);
                        index = largest;
                    } else {
                        break;
                    }
                }
            }
        }
        
        return risultato;
    }
    
    bool empty() const {
        return heap.empty();
    }
    
    int size() const {
        return heap.size();
    }
};

class HashTable {
private:
    vector<MaxHeap> tabella;
    int dimensione;
    
    // Funzione hash: somma dei valori ASCII dei caratteri modulo (N/2)
    int hashFunction(const string& x) {
        int somma = 0;
        for (char c : x) {
            somma += (int)c;
        }
        return somma % (dimensione);
    }

public:
    HashTable(int N) {
        dimensione = N / 2;
        tabella.resize(dimensione);
    }
    
    void inserisciServizio(const string& id, const string& categoria, int gradimento) {
        int indice = hashFunction(id);
        tabella[indice].inserisci(Servizio(id, categoria, gradimento));
    }
    
    void soluzione(int K) {
        map<string, int> conteggioCategorie;
        
        // Per ogni posizione della tabella hash
        for (int i = 0; i < dimensione; i++) {
            if (!tabella[i].empty()) {
                // Estrai i primi K servizi da questo heap
                vector<Servizio> primiK = tabella[i].estraiPrimiK(K);
                
                // Conta le categorie
                for (const Servizio& servizio : primiK) {
                    conteggioCategorie[servizio.categoria]++;
                }
            }
        }
        
        // Trova la categoria con il maggior numero di elementi
        string categoriaMigliore = "";
        int maxConteggio = 0;
        
        for (const auto& coppia : conteggioCategorie) {
            if (coppia.second > maxConteggio || 
                (coppia.second == maxConteggio && (categoriaMigliore.empty() || coppia.first < categoriaMigliore))) {
                categoriaMigliore = coppia.first;
                maxConteggio = coppia.second;
            }
        }
        
        cout << categoriaMigliore << endl;
    }
};

int main() {
    int N, K;
    cin >> N >> K;
    
    HashTable tabella_hash(N);
    
    for (int i = 0; i < N; i++) {
        string id, categoria;
        int gradimento;
        cin >> id >> categoria >> gradimento;
        tabella_hash.inserisciServizio(id, categoria, gradimento);
    }
    
    tabella_hash.soluzione(K);
    
    return 0;
}
```

# Heap 
``` cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

// ==========================
// Struttura nodo/servizio
// ==========================
struct Nodo {
    int valore;          // ID
    string nome;
    string stato;
    int costo;           // costo necessario per raggiungerlo dal padre
    int costo_da_radice; // costo accumulato dalla radice

    Nodo(int v=0, string n="", string s="", int c=0, int r=0) 
        : valore(v), nome(n), stato(s), costo(c), costo_da_radice(r) {}
};

// ==========================
// Classe Heap (max-heap su costo_da_radice)
// ==========================
class Heap {
    vector<Nodo> A;

    // criterio di priorità: maggiore costo_da_radice sopra
    bool priorita(const Nodo& a, const Nodo& b) {
        if (a.costo_da_radice != b.costo_da_radice)
            return a.costo_da_radice > b.costo_da_radice;
        return a.valore < b.valore; // a parità, ID minore prima
    }

    void heapifyUp(int i) {
        while (i > 0) {
            int p = (i - 1) / 2;
            if (priorita(A[i], A[p])) {
                swap(A[i], A[p]);
                i = p;
            } else break;
        }
    }

    void heapifyDown(int i) {
        int n = A.size();
        while (true) {
            int l = 2*i + 1;
            int r = 2*i + 2;
            int best = i;
            if (l < n && priorita(A[l], A[best])) best = l;
            if (r < n && priorita(A[r], A[best])) best = r;
            if (best != i) {
                swap(A[i], A[best]);
                i = best;
            } else break;
        }
    }

public:
    void push(const Nodo& x) {
        A.push_back(x);
        heapifyUp(A.size()-1);
    }

    bool empty() { return A.empty(); }

    Nodo top() { return A.front(); }

    void pop() {
        if (A.empty()) return;
        A[0] = A.back();
        A.pop_back();
        if (!A.empty()) heapifyDown(0);
    }

    // ==========================
    // Soluzione: dato M e città obiettivo C,
    // estraggo i nodi dal heap e raccolgo gli stati
    // che soddisfano i vincoli
    // ==========================
    void soluzione(int M, string C) {
        vector<string> stati;

        Heap temp = *this; // lavoro su una copia del heap
        while (!temp.empty()) {
            Nodo cur = temp.top();
            temp.pop();

            if (cur.nome == C && cur.costo_da_radice <= M) {
                stati.push_back(cur.stato);
            }
        }

        if (stati.empty()) {
            cout << "NO\n";
            return;
        }

        sort(stati.begin(), stati.end());
        for (int i=0; i<stati.size(); i++) {
            if (i == 0 || stati[i] != stati[i-1])
                cout << stati[i] << "\n";
        }
    }
};

int main() {
    int N, M;
    string C;
    cin >> N >> M >> C;

    Heap heap;

    for (int i=0; i<N; i++) {
        int valore, costo;
        string nome, stato;
        cin >> valore >> nome >> stato >> costo;
        // qui costo_da_radice = costo (non avendo un albero con padre)
        heap.push(Nodo(valore, nome, stato, costo, costo));
    }

    heap.soluzione(M, C);

    return 0;
}

```

# Albero generico con liste multiple
``` cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

struct Nodo {
    string valore;
    vector<Nodo*> figli;

    Nodo(string v="") : valore(v) {}
};

class AlberoLista {
    Nodo* radice;

    void stampaPrivato(Nodo* u, int livello) {
        if (!u) return;
        cout << string(livello, '-') << u->valore << "\n";
        for (Nodo* f : u->figli)
            stampaPrivato(f, livello+1);
    }

public:
    AlberoLista() { radice = nullptr; }

    void setRadice(string v) {
        radice = new Nodo(v);
    }

    Nodo* getRadice() { return radice; }

    void aggiungiFiglio(Nodo* padre, string v) {
        padre->figli.push_back(new Nodo(v));
    }

    void stampa() {
        stampaPrivato(radice, 0);
    }
};

```

# Albero generico con figlio-fratello
``` cpp
#include <iostream>
#include <string>
using namespace std;

struct Nodo {
    string valore;
    Nodo* primoFiglio;
    Nodo* fratello;

    Nodo(string v="") : valore(v), primoFiglio(nullptr), fratello(nullptr) {}
};

class AlberoFS {
    Nodo* radice;

    void stampaPrivato(Nodo* u, int livello) {
        if (!u) return;
        cout << string(livello, '-') << u->valore << "\n";
        stampaPrivato(u->primoFiglio, livello+1); // figli
        stampaPrivato(u->fratello, livello);      // fratelli
    }

public:
    AlberoFS() { radice = nullptr; }

    void setRadice(string v) {
        radice = new Nodo(v);
    }

    Nodo* getRadice() { return radice; }

    void aggiungiFiglio(Nodo* padre, string v) {
        Nodo* nuovo = new Nodo(v);
        if (!padre->primoFiglio) {
            padre->primoFiglio = nuovo;
        } else {
            Nodo* cur = padre->primoFiglio;
            while (cur->fratello) cur = cur->fratello;
            cur->fratello = nuovo;
        }
    }

    void stampa() {
        stampaPrivato(radice, 0);
    }
};

```

# Heap + grafi
``` cpp
#include <iostream>
#include <vector>
#include <list>
#include <string>
using namespace std;

struct Arco {
    int to; // indice del nodo di destinazione
    int peso;
    Arco(int t, int p=1) : to(t), peso(p) {}
};

struct Grafo {
    vector<list<Arco>> adj;

    Grafo(int n=0) {
        adj.resize(n);
    }

    void aggiungiArco(int u, int v, int peso=1) {
        adj[u].push_back(Arco(v, peso));
        // se non orientato:
        // adj[v].push_back(Arco(u, peso));
    }

    void stampa() {
        for (int i=0; i<adj.size(); i++) {
            cout << i << ": ";
            for (auto& e : adj[i])
                cout << "(" << e.to << "," << e.peso << ") ";
            cout << "\n";
        }
    }
};

class HashTable {
    vector<Grafo> table;
    int size;

    int hashFunction(int key) {
        return key % size;
    }

public:
    HashTable(int n) : size(n) {
        table.resize(size);
    }

    void inserisci(int key, int v1, int v2, int peso=1) {
        int idx = hashFunction(key);
        int n = max(v1,v2)+1;
        if ((int)table[idx].adj.size() < n)
            table[idx].adj.resize(n);
        table[idx].aggiungiArco(v1, v2, peso);
    }

    void stampa() {
        for (int i=0; i<size; i++) {
            cout << "Bucket " << i << ":\n";
            table[i].stampa();
        }
    }
};

```
# Stampe prova
visita PREORDER (ordine anticipato) per albero binario di ricerca
Guardo prima me e poi i miei figli --> finchè si può si scende a sinistra poi guardo gli alberi destri
``` cpp
void preOrder(Node* tree) {

	if (!tree) return;
	else {
		cout << tree->label;
		preOrder(tree->left);
		preOrder(tree->right);
	}
}
```

visita POSTORDER (ordine differito) per albero binario di ricerca
Guardo prima i figli e poi me --> (parto dalle foglie)
``` cpp
void postOrder(Node* tree) {

if (!tree) return;
else {
	postOrder(tree->left);
	postOrder(tree->right);
	<esamina tree->label>;
	}
}
```

visita INORDER (ordine simmetrico) per albero binario di ricerca
guardo sotto albero sx, poi radice, poi sottoalbero dx
--> ordine crescente
``` cpp
void inOrder(Node* tree) {

if (!tree) return;
	else {
	inOrder(tree->left);
	<esamina tree->label>;
	inOrder(tree-> right);
	}
}
```

stampa albero ruotato di 90 gradi (la radice è a sinistra, il sottoalbero destro sta sopra e il sottoalbero sinistro sta sotto)
``` cpp
// stampa albero in modo "grafico"
void stampaAlbero(Nodo* u, int spazio = 0, int passo = 6) {
    if (u == nullptr) return;

    // aumento distanza per il livello successivo
    spazio += passo;

    // prima stampo il sottoalbero destro
    stampaAlbero(u->destro, spazio, passo);

    // stampo il nodo corrente
    cout << endl;
    for (int i = passo; i < spazio; i++) cout << " ";
    cout << u->valore << "(" << u->lunghezza_L << ")" << endl;

    // poi stampo il sottoalbero sinistro
    stampaAlbero(u->sinistro, spazio, passo);
}
```

``` cpp
// Funzione di utilità per debug alberi
void stampaAlbero() {
	cout << "Struttura ABR:\n";
    stampaRicorsiva(radice, 0);
}
    
void stampaRicorsiva(Nodo* u, int spazio) {
	if (u == nullptr) return;
    spazio += 5;
    stampaRicorsiva(u->destro, spazio);
    cout << endl;
    for (int i = 5; i < spazio; i++) cout << " ";
        cout << u->valore << "(" << (u->pari ? "p" : "d") << ",D=" << u->D_x << ")\n";
        stampaRicorsiva(u->sinistro, spazio);
}
```

stampa tabella hash con liste di trabocco
``` cpp
void stampaTabella() {
    cout << "\n=== HASH TABLE ===\n";
    for (int i = 0; i < size; i++) {
        cout << "[" << i << "] -> ";
        Nodo* cur = table[i];
        if (!cur) {
            cout << " (vuota)";
        }
        while (cur) {
            cout << "(" << cur->ID << "," << cur->cognome << ")";
            if (cur->next) cout << " -> ";
            cur = cur->next;
        }
        cout << "\n";
    }
    cout << "==================\n";
}

```

stampa tabella hash con ABR --> questa versione ti propone i nodi dell'albero in ordine crescente, non di inserimento
``` cpp
// nel public della classe ABR
NodoABR* getRadice() const { return radice; }

// nel public della classe HashTaable
void stampaTabella() {
    cout << "\n=== HASH TABLE ===\n";
    for (int i = 0; i < dimensione; i++) {
        cout << "[" << i << "] -> ";

        NodoABR* root = tabella[i].getRadice();

        if (!root) {
            cout << "(vuota)";
        } else {
            // lambda ricorsiva con auto
            auto stampaBST = [&](auto&& self, NodoABR* nodo) -> void {
                if (!nodo) return;
                self(self, nodo->sinistro);
                cout << "(" << nodo->valore << "," << nodo->stringa << ") ";
                self(self, nodo->destro);
            };
            stampaBST(stampaBST, root);
        }
        cout << "\n";
    }
    cout << "==================\n";
}
```

stampa TabellaHash con ABR con struttura grafica
``` cpp
// nel public di ABR
NodoABR* getRadice() const { return radice; }

//nel public di TabellaHash
void stampaTabella() {
    cout << "\n=== HASH TABLE ===\n";
    for (int i = 0; i < dimensione; i++) {
        cout << "[" << i << "] -> ";

        NodoABR* root = tabella[i].getRadice();

        if (!root) {
            cout << "(vuota)\n";
        } else {
            cout << "\n";
            
            // Funzione ricorsiva per stampare con indentazione
            auto stampaStruttura = [&](auto&& self, NodoABR* nodo, string indent, bool isRight, bool isRoot = false) -> void {
                if (!nodo) return;
                
                cout << indent;
                if (!isRoot) {
                    if (isRight) {
                        cout << "R----";
                        indent += "     ";
                    } else {
                        cout << "L----";
                        indent += "|    ";
                    }
                } else {
                    indent += "     ";
                }
                
                cout << "(" << nodo->valore << "," << nodo->stringa << ")\n";
                
                self(self, nodo->sinistro, indent, false, false);
                self(self, nodo->destro, indent, true, false);
            };
            
            stampaStruttura(stampaStruttura, root, "", false, true);
        }
    }
    cout << "==================\n";
}
```






