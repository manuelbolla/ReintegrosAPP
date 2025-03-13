import { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Table, TableHeader, TableRow, TableHead, TableBody, TableCell } from "@/components/ui/table";
import { db } from "../firebaseConfig";
import { collection, addDoc, getDocs, updateDoc, doc } from "firebase/firestore";

function calcularDiasHabiles(fecha) {
  let dias = 0;
  let fechaActual = new Date(fecha);
  let hoy = new Date();
  while (fechaActual < hoy) {
    fechaActual.setDate(fechaActual.getDate() + 1);
    if (fechaActual.getDay() !== 0 && fechaActual.getDay() !== 6) {
      dias++;
    }
  }
  return dias;
}

export default function ReintegrosApp() {
  const [compras, setCompras] = useState([]);
  const [monto, setMonto] = useState("");
  const [reintegro, setReintegro] = useState("");
  const [comercio, setComercio] = useState("");
  const [fecha, setFecha] = useState(new Date().toISOString().split("T")[0]);

  useEffect(() => {
    const fetchCompras = async () => {
      const querySnapshot = await getDocs(collection(db, "compras"));
      const comprasData = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setCompras(comprasData);
    };
    fetchCompras();
  }, []);

  const agregarCompra = async () => {
    if (!monto || !reintegro || !comercio) return;
    const docRef = await addDoc(collection(db, "compras"), { monto, reintegro, comercio, fecha, estado: "Pendiente" });
    setCompras([...compras, { id: docRef.id, monto, reintegro, comercio, fecha, estado: "Pendiente" }]);
    setMonto("");
    setReintegro("");
    setComercio("");
    setFecha(new Date().toISOString().split("T")[0]);
  };

  const marcarAcreditado = async (id) => {
    const compraRef = doc(db, "compras", id);
    await updateDoc(compraRef, { estado: "Acreditado" });
    setCompras(compras.map(compra => (compra.id === id ? { ...compra, estado: "Acreditado" } : compra)));
  };

  return (
    <div className="p-6 max-w-xl mx-auto">
      <h1 className="text-xl font-bold mb-4">Seguimiento de Reintegros</h1>
      <Card className="mb-4">
        <CardContent className="space-y-2">
          <Input placeholder="Nombre del comercio" value={comercio} onChange={(e) => setComercio(e.target.value)} />
          <Input type="date" value={fecha} onChange={(e) => setFecha(e.target.value)} />
          <Input placeholder="Monto de compra" value={monto} onChange={(e) => setMonto(e.target.value)} />
          <Input placeholder="Reintegro esperado" value={reintegro} onChange={(e) => setReintegro(e.target.value)} />
          <Button onClick={agregarCompra}>Agregar Compra</Button>
        </CardContent>
      </Card>
      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>Comercio</TableHead>
            <TableHead>Fecha</TableHead>
            <TableHead>Monto</TableHead>
            <TableHead>Reintegro</TableHead>
            <TableHead>Estado</TableHead>
            <TableHead>Acción</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {compras.map((compra) => {
            const diasHabiles = calcularDiasHabiles(compra.fecha);
            const vencido = compra.estado === "Pendiente" && diasHabiles > 8;
            return (
              <TableRow key={compra.id} className={vencido ? "bg-red-200" : ""}>
                <TableCell>{compra.comercio}</TableCell>
                <TableCell>{compra.fecha}</TableCell>
                <TableCell>{compra.monto}</TableCell>
                <TableCell>{compra.reintegro}</TableCell>
                <TableCell>{compra.estado}</TableCell>
                <TableCell>
                  {compra.estado === "Pendiente" && (
                    <Button size="sm" onClick={() => marcarAcreditado(compra.id)}>
                      Marcar Acreditado
                    </Button>
                  )}
                </TableCell>
              </TableRow>
            );
          })}
        </TableBody>
      </Table>
    </div>
  );
}
